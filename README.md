## 🔄 `root.tsx`

```diff
+ import {XoBuilder} from '@xotiny/xb-react-elements';
+ import wcJs from '~/wc/wc.js?url';
...

+ <XoBuilder.Root>
  <html lang={locale.language}>
+   <XoBuilder.Root.Head jsUrls={[wcJs]}>
-    <head>
      <meta charSet="utf-8" />
      <meta name="viewport" content="width=device-width,initial-scale=1" />
      <meta
        name="msvalidate.01"
        content="A352E6A0AF9A652267361BBB572B8468"
      />
      <link rel="stylesheet" href={styles}></link>
      <Meta />
      <Links />
-    </head>
+   </XoBuilder.Root.Head>
    <body>
      {data ? (
        <Analytics.Provider
          cart={data.cart}
          shop={data.shop}
          consent={data.consent}
        >
          <PageLayout
            key={`${locale.language}-${locale.country}`}
            layout={data.layout}
          >
            {children}
          </PageLayout>
        </Analytics.Provider>
      ) : (
        children
      )}
      <ScrollRestoration nonce={nonce} />
      <Scripts nonce={nonce} />
    </body>
  </html>
+ </XoBuilder.Root>
```

## 🔄 `entry.server.tsx`

```diff
+ import {XoBuilder} from '@xotiny/xb-react-elements';

...

+ const {nonce, header, NonceProvider} = XoBuilder.createContentSecurityPolicy({
- const {nonce, header, NonceProvider} = createContentSecurityPolicy({
  shop: {
    checkoutDomain: context.env.PUBLIC_CHECKOUT_DOMAIN,
    storeDomain: context.env.PUBLIC_STORE_DOMAIN,
  },
  scriptSrc: [
    'self',
    'https://cdn.shopify.com',
    'https://shopify.com',
    'https://www.google-analytics.com',
    'https://www.googletagmanager.com',
    ...(process.env.NODE_ENV !== 'production' ? ['http://localhost:*'] : []),
  ],
});
```

## 🔄 `($locale)._index.tsx`

```diff
  import {useLoaderData, type MetaFunction} from '@remix-run/react';
import type {LoaderFunctionArgs} from '@remix-run/server-runtime';
import {defer} from '@remix-run/server-runtime';
import {XoBuilder} from '@xotiny/xb-react-elements';

import {seoPayload} from '~/lib/seo.server';

export async function loader(args: LoaderFunctionArgs) {
  const {params, context, request} = args;
  const {language, country} = context.storefront.i18n;

  if (
    params.locale &&
    params.locale.toLowerCase() !== `${language}-${country}`.toLowerCase()
  ) {
    // If the locale URL param is defined, yet we still are on `EN-US`
    // the the locale param must be invalid, send to the 404 page
    throw new Response(null, {status: 404});
  }

  // Await the critical data required to render initial state of the page
  const criticalData = await XoBuilder.loadPageData({
    pageType: 'home',
    args,
  });

  const seo = {
    ...seoPayload.home({url: request.url}),
    ...criticalData.metaData,
  };

  return defer({...criticalData, seo});
}

export const meta: MetaFunction<typeof loader> = (metaData) => {
  return XoBuilder.pageMeta(metaData);
};

export default function Homepage() {
  const {pageData, shopifyData, cssContent} = useLoaderData<typeof loader>();

  return (
    <XoBuilder.Layout
      isDev={process.env.NODE_ENV === 'development'}
      page={pageData}
      shopifyData={shopifyData}
      cssContent={cssContent}
    />
  );
}

```