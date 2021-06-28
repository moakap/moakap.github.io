---
title: React Navigation中的登录认证流程
date: 2021-06-28 16:45:51
tags:
  - React
  - React Navigation
  - Authentication
categories:
  - Technology
  - React
---

大部分应用需要某种方式的用户认证，以便用户可以访问个人相关的数据或其它隐私数据。典型的流程是这样的：

- 用户打开app
- app从加密的持久存储中加载认证信息
- 加载完以后，根据用户认证状态，用户会跳转到认证页面，或者到app主页面。
- 如果用户退出登录，我们清理用户的认证信息，并且跳转回认证页面。

注意：这里说的认证页面，就是指跟登录相关的页面，比如登录、注册、忘记密码等。

<!-- more -->

## 想要的结果

我们想要的认证流程：当用户登录完以后，我们就要完全抛弃认证状态相关的流程，并且清楚对应的认证页面。及时用户通过物理键后退，我们希望用户不会再次跳转到认证流程。

## 如何实现

我们可以基于不同的条件定义不同的页面。例如，如果用户已经登录，我们定义Home, Profile, Settings等页面。如果用户没登录，定义SignIn和SignUp等页面。

例如：

```jsx
isSignedIn ? (
  <>
    <Stack.Screen name="Home" component={HomeScreen} />
    <Stack.Screen name="Profile" component={ProfileScreen} />
    <Stack.Screen name="Settings" component={SettingsScreen} />
  </>
) : (
  <>
    <Stack.Screen name="SignIn" component={SignInScreen} />
    <Stack.Screen name="SignUp" component={SignUpScreen} />
  </>
)
```

像上边这样，如果isSignedIn是true, React Navigation只能看到Home, Profile, Settings页面；而当isSignedIn是false时，只能看到SignIn和SignUp两个页面。这就保证了用户没有登录时，无论如何不会跳转到Home, Progile和Settings等页面；而用户登录后，也不会跳转到SignIn和SignUp等页面。

这种模式已经被其它路由库（比如Ract Router）使用了很长时间，也被称作“受保护路由”。这里，那些需要用户登录的页面被“保护”着，如果用户没有登录，就无法导航到这些页面。

所有行为会随着IsSignedIn的值变化而改变。这里我们假设isSignedIn初始值为false. 也就意味着，要么SignIn或SignUp会显示。当用户登录以后，isSignedIn的值变成true。这时从React Navigation角度来看，SignIn和SignUp是未定义的，react navigation会将他们从导航栈中删除。然后会显示Home页面，因为你isSignedIn为true时Home页面为导航栈里的第一个页面。

这里是以导航栈为例来说的，但是同样的逻辑同样适用于其他导航器。

通过变量的不同值来定义不同的页面，我们可以用很简单的方式实现认证流程，整个过程不需要额外的逻辑来处理页面的显示。

## 条件渲染页面时，不要手动跳转

很重要的一点，如果使用了上边例子中的方式，就不要通过navigation.navigete('Home')或其它手工方式跳转到Home页面。当isSigned值变化时，React Navigation会自动实现页面跳转  - 当isSignedIn变成true时显示Home页面，为false时跳转到SignIn页面。如果使用了手工方式导航页面，会产生错误。

## 页面定义

在我们的导航器中，我们可以根据不同条件定义不同页面。这里我们假设有3个页面：

- SplashScreen - 加载token时的启动或加载页面。
- SignedInScreen - 用户没有登录时我们要显示的登录页面。
- HomeScreen - 用户登录以后要显示的主页。

所以我们的导航器是这样的:

```jsx
if (state.isLoading) {
  // We haven't finished checking for the token yet
  return <SplashScreen />;
}

return (
  <Stack.Navigator>
    {state.userToken == null ? (
      // No token found, user isn't signed in
      <Stack.Screen
        name="SignIn"
        component={SignInScreen}
        options={{
          title: 'Sign in',
          // When logging out, a pop animation feels intuitive
          // You can remove this if you want the default 'push' animation
          animationTypeForReplace: state.isSignout ? 'pop' : 'push',
        }}
      />
    ) : (
      // User is signed in
      <Stack.Screen name="Home" component={HomeScreen} />
    )}
  </Stack.Navigator>
);
```

在上边的代码片段中，isLoading表示我们仍然在检测token是否存在。一般会检测SecureStore里是否有token，并且验证token的合法性。拿到token并且验证以后，我们需要设置userToken. 我们也可以有另外一个状态isSignout来定义登出时的不同效果。

这里需要重点关注的一点就是，我们根据state变量来定义不同的页面。

- SignIn页面只有在userToken为null（用户未登录）时才被定义。
- Home页面只有在userToken非null（用户已登录）时才被定义。

这里我们根据条件为每种情况只定义了一个页面。但是你也可以定义多页页面。例如，用户未登录时我们可能还需要定义重置密码、注册等页面。同样的，登录以后也不只一个页面。我们可以用React.Fragment来定义多个页面：

```jsx
state.userToken == null ? (
  <>
    <Stack.Screen name="SignIn" component={SignInScreen} />
    <Stack.Screen name="SignUp" component={SignUpScreen} />
    <Stack.Screen name="ResetPassword" component={ResetPassword} />
  </>
) : (
  <>
    <Stack.Screen name="Home" component={HomeScreen} />
    <Stack.Screen name="Profile" component={ProfileScreen} />
  </>
);
```

注意，如果在栈导航器中既有登录相关页面，也有与登录不相关的页面，我们推荐使用一个导航栈，然后在导航栈中根据条件来定义不同页面，而不是根据条件定义两个不同的导航器。这样可以实现登录和登出之间页面的跳转。

## 还原token的实现逻辑

注意：下边只是一个在app中实现认证逻辑的例子，你不需要完全准守其中的流程。

从之前的代码段，可以看到我们需要3个state变量

- isLoading - 当正在检查SecureStore中是否有token时设置为true。
- isSignout - 用户登出时设置为true，否则为false。
- userToken - 用户token。如果为非null时我们假设用户已经登录，否则未登录。

因此接下来我们需要：

- 添加获取token的逻辑，以及登录和登出的逻辑
- 为其它组件导出登录和登出的方法

这个教程里我们将使用React.useReducer和React.useContext。如果你已经在使用其它状态管理库（比如Redux或Mobx），你可以使用对应的方法实现对应的功能。事实上，在一个大点的app中，全局状态管理库更适合用来保存认证相关的token。同样的方法也适用于这些状态管理库。

首先我们为auth创建一个context：

```jsx
import * as React from 'react';

const AuthContext = React.createContext();
```

然后我们的组件是这样的：

```jsx
import * as React from 'react';
import * as SecureStore from 'expo-secure-store';

export default function App({ navigation }) {
  const [state, dispatch] = React.useReducer(
    (prevState, action) => {
      switch (action.type) {
        case 'RESTORE_TOKEN':
          return {
            ...prevState,
            userToken: action.token,
            isLoading: false,
          };
        case 'SIGN_IN':
          return {
            ...prevState,
            isSignout: false,
            userToken: action.token,
          };
        case 'SIGN_OUT':
          return {
            ...prevState,
            isSignout: true,
            userToken: null,
          };
      }
    },
    {
      isLoading: true,
      isSignout: false,
      userToken: null,
    }
  );

  React.useEffect(() => {
    // Fetch the token from storage then navigate to our appropriate place
    const bootstrapAsync = async () => {
      let userToken;

      try {
        userToken = await SecureStore.getItemAsync('userToken');
      } catch (e) {
        // Restoring token failed
      }

      // After restoring token, we may need to validate it in production apps

      // This will switch to the App screen or Auth screen and this loading
      // screen will be unmounted and thrown away.
      dispatch({ type: 'RESTORE_TOKEN', token: userToken });
    };

    bootstrapAsync();
  }, []);

  const authContext = React.useMemo(
    () => ({
      signIn: async data => {
        // In a production app, we need to send some data (usually username, password) to server and get a token
        // We will also need to handle errors if sign in failed
        // After getting token, we need to persist the token using `SecureStore`
        // In the example, we'll use a dummy token

        dispatch({ type: 'SIGN_IN', token: 'dummy-auth-token' });
      },
      signOut: () => dispatch({ type: 'SIGN_OUT' }),
      signUp: async data => {
        // In a production app, we need to send user data to server and get a token
        // We will also need to handle errors if sign up failed
        // After getting token, we need to persist the token using `SecureStore`
        // In the example, we'll use a dummy token

        dispatch({ type: 'SIGN_IN', token: 'dummy-auth-token' });
      },
    }),
    []
  );

  return (
    <AuthContext.Provider value={authContext}>
      <Stack.Navigator>
        {state.userToken == null ? (
          <Stack.Screen name="SignIn" component={SignInScreen} />
        ) : (
          <Stack.Screen name="Home" component={HomeScreen} />
        )}
      </Stack.Navigator>
    </AuthContext.Provider>
  );
}
```

## 完成其它组件

在认证页面怎么实现文本输入和按钮认证不是这里要讨论的范围，这里只是简单放置一些占位函数。

```jsx
function SignInScreen() {
  const [username, setUsername] = React.useState('');
  const [password, setPassword] = React.useState('');

  const { signIn } = React.useContext(AuthContext);

  return (
    <View>
      <TextInput
        placeholder="Username"
        value={username}
        onChangeText={setUsername}
      />
      <TextInput
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      <Button title="Sign in" onPress={() => signIn({ username, password })} />
    </View>
  );
}
```

