---
template: BlogPost
path: /using-apolloclient-for-real
date: 2020-06-15T14:05:49.332Z
title: Using ApolloClient in a real project
metaDescription: >-
  If you have been using ApolloClient in your projects, you have probably
  started using their "apollo-boost" package.
thumbnail: /assets/image-7.jpg
---
If you have been using ApolloClient in your projects, you have probably started using their *"apollo-boost"* package. And for starting this is the right approach, but the limitations of that setup appear very quickly when working on a production application. Something as simple as using a graphQL server that requires authentication causes a steep learning curve into the inner workings of ApolloClient. My goal is to point out some of the stumbling blocks I have been encountering and the links to the solutions or articles that helped me.

The Migration: If you'd like to use subscriptions, swap out the Apollo cache, or add an existing Apollo Link to your network stack that isn't already included, you will have to set Apollo Client up manually. Their guide (https://www.apollographql.com/docs/react/migrating/boost-migration/) is very well written. This will help you get the right packages installed in your project. However...

This setup for authentication may not work or give you the flexibility needed to tie to your backend server. A middleware function needs to be created (https://www.apollographql.com/docs/react/networking/network-layer/#middleware). A combination of those links will help you get a proper migration from boost to a real-world set up of ApolloClient. Bellow is what a completed set up will look like.

```javascript
import { ApolloClient } from 'apollo-client';
import { ApolloLink } from 'apollo-link';
import { HttpLink } from 'apollo-link-http';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { onError } from 'apollo-link-error';
import LogRocket from 'logrocket';
import { RetryLink } from 'apollo-link-retry';
import { getConfig } from './helpers/config-util';
import { getStore } from './helpers/store-util';

const authMiddleware = new ApolloLink((operation, forward) => {
	// add the authorization to the headers
	// https://www.apollographql.com/docs/react/networking/network-layer/#middleware
	const token = getStore()?.getState()?.auth0?.token;
	operation.setContext(({ headers = {} }) => ({
		headers: {
			...headers,
			authorization: `Bearer ${token}`,
		},
	}));
	return forward(operation);
});
// Adding fix to improve logRocket recording
// https://docs.logrocket.com/docs/troubleshooting-sessions#apollo-client

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
				LogRocket.captureException(networkError);
				console.error(`[Network error]:`, networkError);
			}
		}),
		authMiddleware,
		new RetryLink(),
		new HttpLink({
			uri: `${getConfig().apiUrl}`,
			fetch: fetcher,
		}),
	]),
	cache: new InMemoryCache(),
});

export default client;
```

P.S. - If the backend has a basic setup, the *authorization* header is not a standard header, so it may throw a CORS error. Make sure you have the server allows that header.

This middleware is touching on the concepts of Apollolinks. This will be the topic of the next post in this series. Since they are their own complex, but powerful feature of ApolloClient
