# react-netlify-identity

> Netlify Identity + React Hooks, with Typescript

[![NPM](https://img.shields.io/npm/v/react-netlify-identity.svg)](https://www.npmjs.com/package/react-netlify-identity) [![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)

Use [Netlify Identity](https://www.netlify.com/docs/identity/) easier with React! This is a thin wrapper over the [gotrue-js](https://github.com/netlify/gotrue-js) library for easily accessing Netlify Identity functionality in your app, with React Hooks. Types are provided.

You can find [a full demo here](https://netlify-gotrue-in-react.netlify.com/) with [source code](https://github.com/netlify/create-react-app-lambda/tree/reachRouterAndGoTrueDemo/src).

**This library is not officially maintained by Netlify.** This is written by swyx for his own use (and others with like minds 😎) and will be maintained as a personal project unless formally adopted by Netlify. See below for official alternatives.

## List of Alternatives

**Lowest level JS Library**: If you want to use the official Javascript bindings to GoTrue, Netlify's underlying Identity service written in Go, use https://github.com/netlify/gotrue-js

**React bindings**: If you want a thin wrapper over Gotrue-js for React, `react-netlify-identity` is a "headless" library, meaning there is no UI exported and you will write your own UI to work with the authentication. https://github.com/sw-yx/react-netlify-identity

**High level overlay**: If you want a "widget" overlay that gives you a nice UI out of the box, with a somewhat larger bundle, check https://github.com/netlify/netlify-identity-widget

**High level popup**: If you want a popup window approach also with a nice UI out of the box, and don't mind the popup flow, check https://github.com/netlify/netlify-auth-providers

## Typescript

Library is written in Typescript. File an issue if you find any problems.

## Install

```bash
yarn add react-netlify-identity
```

## Usage

⚠️ **Important:** You will need to have an active Netlify site running with Netlify Identity turned on. [Click here for instructions](https://www.netlify.com/docs/identity/#getting-started) to get started/double check that it is on. We will need your site's url (e.g. `https://mysite.netlify.com`) to initialize `useNetlifyIdentity`.

**As a React Hook**, you can destructure these variables and methods:

- `user: User`
- `setUser`: directly set the user object. Not advised; use carefully!! mostly you should use the methods below
- `isConfirmedUser: boolean`: if they have confirmed their email
- `isLoggedIn: boolean`: if the user is logged in
- `signupUser(email: string, password: string, data: Object)`
- `loginUser(email: string, password: string)`
- `logoutUser()`
- `requestPasswordRecovery(email: string)`
- `recoverAccount(token: string, remember?: boolean | undefined)`
- `updateUser(fields: Object)`
- `getFreshJWT()`
- `authedFetch(endpoint: string, obj = {})` (a thin axios-like wrapper over `fetch` that has the user's JWT attached, for convenience pinging Netlify Functions with Netlify Identity)

```tsx
import * as React from 'react';

import { useNetlifyIdentity } from 'react-netlify-identity';

const IdentityContext = React.createContext(); // not necessary but recommended
function App() {
  const identity = useNetlifyIdentity(url); // supply the url of your Netlify site instance. VERY IMPORTANT
  return (
    <IdentityContext.Provider value={identity}>
      {/* rest of your app */}
    </IdentityContext.Provider>
  );
}
```

<details>
<summary>
<h3 style="color: red">
Click for More Example code
</h3>
</summary>

```tsx
// log in/sign up example
function Login() {
  const { loginUser, signupUser } = React.useContext(IdentityContext);
  const formRef = React.useRef();
  const [msg, setMsg] = React.useState('');
  const signup = () => {
    const email = formRef.current.email.value;
    const password = formRef.current.password.value;
    signupUser(email, password)
      .then(user => {
        console.log('Success! Signed up', user);
        navigate('/dashboard');
      })
      .catch(err => console.error(err) || setMsg('Error: ' + err.message));
  };
  return (
    <form
      ref={formRef}
      onSubmit={e => {
        e.preventDefault();
        const email = e.target.email.value;
        const password = e.target.password.value;
        load(loginUser(email, password))
          .then(user => {
            console.log('Success! Logged in', user);
            navigate('/dashboard');
          })
          .catch(err => console.error(err) || setMsg('Error: ' + err.message));
      }}
    >
      <div>
        <label>
          Email:
          <input type="email" name="email" />
        </label>
      </div>
      <div>
        <label>
          Password:
          <input type="password" name="password" />
        </label>
      </div>
      <div>
        <input type="submit" value="Log in" />
        <button onClick={signup}>Sign Up </button>
        {msg && <pre>{msg}</pre>}
      </div>
    </form>
  );
}

// log out user
function Logout() {
  const { logoutUser } = React.useContext(IdentityContext);
  return <button onClick={logoutUser}>You are signed in. Log Out</button>;
}

// check `identity.user` in a protected route
function PrivateRoute(props) {
  const identity = React.useContext(IdentityContext);
  let { as: Comp, ...rest } = props;
  return identity.user ? (
    <Comp {...rest} />
  ) : (
    <div>
      <h3>You are trying to view a protected page. Please log in</h3>
      <Login />
    </div>
  );
}

// check if user has confirmed their email
// use authedFetch API to make a request to Netlify Function with the user's JWT token,
// letting your function use the `user` object
function Dashboard() {
  const { isConfirmedUser, authedFetch } = React.useContext(IdentityContext);
  const [msg, setMsg] = React.useState('Click to load something');
  const handler = () => {
    authedFetch.get('/.netlify/functions/authEndPoint').then(setMsg);
  };
  return (
    <div>
      <h3>This is a Protected Dashboard!</h3>
      {!isConfirmedUser && (
        <pre style={{ backgroundColor: 'papayawhip' }}>
          You have not confirmed your email. Please confirm it before you ping
          the API.
        </pre>
      )}
      <hr />
      <div>
        <p>You can try pinging our authenticated API here.</p>
        <p>
          If you are logged in, you should be able to see a `user` info here.
        </p>
        <button onClick={handler}>Ping authenticated API</button>
        <pre>{JSON.stringify(msg, null, 2)}</pre>
      </div>
    </div>
  );
}
```

</details>

**As a render prop**: This is also exported as a render prop component, `NetlifyIdentity`, but we're not quite sure if its that useful if you can already use hooks in your app:

```tsx
<NetlifyIdentity domain="https://mydomain.netlify.com">
  {({ loginUser, signupUser }) => {
    // use it
  }}
</NetlifyIdentity>
```

## License

MIT © [sw-yx](https://github.com/sw-yx)
