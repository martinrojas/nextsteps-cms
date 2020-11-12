---
template: BlogPost
path: /apollo-client-graphQL-and-auth
date: 2020-09-02T00:56:57.613Z
title: Apollo client graphQL and Auth0 - A complete implementation
metaDescription: >-
  This article has come together after months of trying different
  implementations and discovering more layers to using Auth0 and Apollo.
thumbnail: /assets/banner.png
---
This article has come together after months of trying different implementations and discovering more layers to using Auth0 and Apollo. Even though I am sure that some of the principles will work well with other similar libraries. I don’t want to take credit for this approach since it was pulled together from multiple forums and GitHub issues and articles.

For this code, I am using the relatively new [auth0-react](https://github.com/auth0/auth0-react) library, but this solution can be used with their auth0-spa SDK. In trying to use an authenticated graphQL server with apollo client / auth0 /react based on the tutorials one of the issues that never seemed to be addressed was a clean way of getting the tokens and if the tokens were expired to seamlessly update the token and retry the query/mutation.

Most of the solutions seemed to be pulling the token from local storage [authentication](https://www.apollographql.com/docs/react/networking/authentication/), but if you had an expired token the only solution offered seemed to delete the expired token and logout the user. The initial break came from `mattwilson1024` in the [auth0 forum](https://community.auth0.com/t/how-to-use-react-auth0-spa-with-graphql/30516).

`AuthorizedApolloProvider.tsx`

```javascript
import { ApolloClient, ApolloProvider, createHttpLink, InMemoryCache } from '@apollo/client';
import { setContext } from '@apollo/link-context';
import React from 'react';

import { useAuth0 } from '../react-auth0-spa';

const AuthorizedApolloProvider = ({ children }) => {
  const { getTokenSilently } = useAuth0();

  const httpLink = createHttpLink({
    uri: 'http://localhost:4000/graphql', // your URI here...
  });

  const authLink = setContext(async () => {
    const token = await getTokenSilently();
    return {
      headers: {
        Authorization: `Bearer ${token}`
      }
    };
  });

  const apolloClient = new ApolloClient({
    link: authLink.concat(httpLink),
    cache: new InMemoryCache(),
    connectToDevTools: true
  });

  return (
    <ApolloProvider client={apolloClient}>
      {children}
    </ApolloProvider>
  );
};

export default AuthorizedApolloProvider;
```

By creating a React Component around the Apollo provider all of the React Hooks and functions become available. Therefore getting the token from the Auth0 hook will mean that it will always be a working token and in the cases that a stored token has expired, it will be the Auth0 library responsible for refreshing the token and not Apollo.

Now based on the apollo documentation the correct way to create add a header by creating a [middleware link](https://www.apollographql.com/docs/react/v2.6/networking/network-layer/#middleware) this, however, is not a function was works with async and therefore had to switch to using the `setContext` link. https://www.apollographql.com/docs/link/links/context/

The problem with this is that if you are passing other header attributes this will not let them go through and apollo setContext documentation doesn’t mention how to get the headers in a call it was from https://github.com/apollographql/apollo-client/issues/4990 that someone had the correct syntax to access the headers.

The final implementation of `AuthorizedApolloProvider` that will allow for extra headers passed from each query also implemented other useful links. Such as a small fix if using logRocket:

```javascript
import { ApolloClient } from 'apollo-client';
import { ApolloLink } from 'apollo-link';
import { ApolloProvider } from 'react-apollo';
import { BatchHttpLink } from 'apollo-link-batch-http';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { onError } from 'apollo-link-error';
import { RetryLink } from 'apollo-link-retry';
import { useAuth0 } from '@auth0/auth0-react';
import LogRocket from 'logrocket';
import React from 'react';
import { setContext } from 'apollo-link-context';

// IF you want to enable/disable dev tools in different enviroments
const devTools = localStorage.getItem('apolloDevTools') || false;

const AuthorizedApolloProvider = ({ children }) => {
	const { getAccessTokenSilently } = useAuth0();
	const authMiddleware = setContext(async (_, { headers, ...context }) => {
		const token = await getAccessTokenSilently();
//Optional if the ti
		if (typeof Storage !== 'undefined') {
			localStorage.setItem('token', token);
		}

		console.log('Network ID:', activeNetworkID);
		return {
			headers: {
				...headers,
				...(token ? { Authorization: `Bearer ${token}` } : {}),
			},
			...context,
		};
	});

	/**
	 * Adding fix to improve logRocket recording
	 * https://docs.logrocket.com/docs/troubleshooting-sessions#apollo-client
	 */

	const fetcher = (...args) => {
		return window.fetch(...args);
	};

	const client = new ApolloClient({
		link: ApolloLink.from([
			onError(({ graphQLErrors, networkError }) => {
				if (graphQLErrors) {
					LogRocket.captureException(graphQLErrors);
					graphQLErrors.forEach(({ message, locations, path }) =>
						console.error(
							`[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`
						)
					);
				}
				if (networkError) {
					// localStorage.removeItem('token');
					LogRocket.captureException(networkError);
					console.error(`[Network error]:`, networkError);
				}
			}),
			authMiddleware,
			new RetryLink(),
			new BatchHttpLink({
				uri: `${getConfig().apiUrl}`,
				fetch: fetcher,
			}),
		]),
		cache: new InMemoryCache(),
		connectToDevTools: devTools,
	});

	return <ApolloProvider client={client}>{children}</ApolloProvider>;
};

export default AuthorizedApolloProvider;
```
