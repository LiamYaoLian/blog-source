---
title: React Performance
date: 2022-12-21 18:21:22
tags:
---
# useCallback
* memoize a callback based on dependency array.
* My example:
```typescript
import qs from 'qs';
import * as auth from 'auth-provider';
import {useAuth} from '../context/auth-context';
import {useCallback} from 'react';

const apiUrl = process.env.REACT_APP_API_URL;

interface Config extends RequestInit {
  token?: string;
  data?: object;
}

/*
* sends an HTTP request; logs out if "401: Unauthorized"
* */
export const http = async (
  endpoint: string,
  {data, token, headers, ...customConfig}: Config = {}
) => {
  const config = {
    // default method: 'GET'
    method: 'GET',
    headers: {
      Authorization: token ? `Bearer ${token}` : '',
      'Content-Type': data ? 'application/json' : '',
    },
    // may override the default method
    ...customConfig,
  };

  if (config.method.toUpperCase() === 'GET') {
    endpoint += `?${qs.stringify(data)}`;
  } else {
    config.body = JSON.stringify(data || {});
  }

  return window
    .fetch(`${apiUrl}/${endpoint}`, config)
    .then(async (response) => {
      // 401: Unauthorized
      if (response.status === 401) {
        await auth.logout();
        window.location.reload();
        return Promise.reject({message: 'Unauthorized.'});
      }

      const data = await response.json();
      if (response.ok) {
        return data;
      } else {
        return Promise.reject(data);
      }
    });
};

/*
* returns a function that can send an HTTP request with config and user token
* */
export const useHttp = () => {
  const {user} = useAuth();
  return useCallback((...[endpoint, config]: Parameters<typeof http>) =>
    http(endpoint, {...config, token: user?.token}), [user?.token]);
};

```

# useMemo
* memoize the return value
`const value = useMemo(() => slowFunction(x, y), [x, y])`

# React.memo
* memoize a component
```typescript
const AllocationTable = (props: AllocationTableProps) => {
  // omit codes
}

const isEqual(oldProps: AllocationTableProps, newProps: AllocationTableProps) {
  return oldProps.startDate === newProps.startDate
  && oldProps.endDate === newProps.endDate;
}

const MemoizedAllocationTable = React.memo(AllocationTable, isEqual);
```

# React.lazy
* import when using
* My example:
```typescript
const AuthenticatedApp = React.lazy(() => import("authenticated-app"));
const UnauthenticatedApp = React.lazy(() => import("unauthenticated-app"));

function App() {
  const {user} = useAuth();

  return (
    <div className='App'>

        <ErrorBoundary fallbackRender={FullPageErrorFallback}>
          {/*React 16.6 added a <Suspense> component that lets you “wait” for some code to load and declaratively
        specify a loading state (like a spinner) while we’re waiting*/}
          {/*React.Suspense: experimental feature*/}
          <React.Suspense fallback={<FullPageLoading/>}>
            {user ? <AuthenticatedApp/>) : <UnauthenticatedApp/>}
          </React.Suspense>
        </ErrorBoundary>
    </div>
  );
}

export default App;
```
