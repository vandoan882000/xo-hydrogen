# Hydrogen template: Demo Store

## Getting started

```bash
npm i @xotiny/xb-react-elements@latest
```

## Modify the files for integration

### `root.tsx`

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

### `entry.server.tsx`

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

### `($locale)._index.tsx`

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

### `($locale).products.$handle.tsx`

```diff
  import {useLoaderData, type MetaFunction} from '@remix-run/react';
import type {LoaderFunctionArgs} from '@remix-run/server-runtime';
import {defer} from '@remix-run/server-runtime';
import {Analytics, useSelectedOptionInUrlParam} from '@shopify/hydrogen';
import {XoBuilder} from '@xotiny/xb-react-elements';
import invariant from 'tiny-invariant';

import {seoPayload} from '~/lib/seo.server';

export async function loader(args: LoaderFunctionArgs) {
  const {params, request} = args;
  const {handle} = params;

  invariant(handle, 'Missing handle param, check route filename');

  // Await the critical data required to render initial state of the page
  const criticalData = await XoBuilder.loadPageData({
    pageType: 'product',
    args,
  });

  const {shopifyData} = criticalData;
  const {productDetail} = shopifyData;

  if (!productDetail) {
    throw new Response(null, {status: 404});
  }

  const seo = {
    ...seoPayload.product({
      product: productDetail,
      selectedVariant: productDetail.selectedOrFirstAvailableVariant,
      url: request.url,
    }),
    ...criticalData.metaData,
  };

  return defer({...criticalData, seo});
}

export const meta: MetaFunction<typeof loader> = (data) => {
  return XoBuilder.pageMeta(data);
};

export default function Product() {
  const {pageData, shopifyData, cssContent} = useLoaderData<typeof loader>();
  const {productDetail} = shopifyData;

  useSelectedOptionInUrlParam(
    productDetail.selectedOrFirstAvailableVariant.selectedOptions,
  );

  return (
    <>
      <XoBuilder.Layout
        isDev={process.env.NODE_ENV === 'development'}
        page={pageData}
        shopifyData={shopifyData}
        cssContent={cssContent}
      />
      <Analytics.ProductView
        data={{
          products: [
            {
              id: productDetail.id,
              title: productDetail.title,
              price:
                productDetail.selectedOrFirstAvailableVariant?.price.amount ||
                '0',
              vendor: productDetail.vendor,
              variantId:
                productDetail.selectedOrFirstAvailableVariant?.id || '',
              variantTitle:
                productDetail.selectedOrFirstAvailableVariant?.title || '',
              quantity: 1,
            },
          ],
        }}
      />
    </>
  );
}

```

### `($locale).collections.$handle.tsx`

```diff
  import {useLoaderData, type MetaFunction} from '@remix-run/react';
import type {LoaderFunctionArgs} from '@remix-run/server-runtime';
import {defer} from '@remix-run/server-runtime';
import {Analytics} from '@shopify/hydrogen';
import {XoBuilder} from '@xotiny/xb-react-elements';
import invariant from 'tiny-invariant';

import {seoPayload} from '~/lib/seo.server';

export const handle = {
  breadcrumb: 'Collection',
};

export async function loader(args: LoaderFunctionArgs) {
  const {params, request} = args;
  const {handle} = params;

  invariant(handle, 'Missing handle param');

  // Await the critical data required to render initial state of the page
  const criticalData = await XoBuilder.loadPageData({
    pageType: 'collection',
    args,
  });

  const {shopifyData} = criticalData;
  const {collectionDetail} = shopifyData;

  if (!collectionDetail) {
    throw new Response(null, {status: 404});
  }

  const seo = {
    ...seoPayload.collection({
      collection: collectionDetail.collection,
      url: request.url,
    }),
    ...criticalData.metaData,
  };

  return defer({...criticalData, seo});
}

export const meta: MetaFunction<typeof loader> = (data) => {
  return XoBuilder.pageMeta(data);
};

export default function Collection() {
  const {pageData, shopifyData, cssContent} = useLoaderData<typeof loader>();
  const {collectionDetail} = shopifyData;

  return (
    <>
      <XoBuilder.Layout
        isDev={process.env.NODE_ENV === 'development'}
        page={pageData}
        shopifyData={shopifyData}
        cssContent={cssContent}
      />
      <Analytics.CollectionView
        data={{
          collection: {
            id: collectionDetail?.collection?.id,
            handle: collectionDetail?.collection?.handle,
          },
        }}
      />
    </>
  );
}

```


### `($locale).pages.$handle.tsx`

```diff
  import type {MetaFunction} from '@remix-run/react';
import {useLoaderData} from '@remix-run/react';
import {defer, type LoaderFunctionArgs} from '@shopify/remix-oxygen';
import {XoBuilder} from '@xotiny/xb-react-elements';
import invariant from 'tiny-invariant';

import {routeHeaders} from '~/data/cache';
import {seoPayload} from '~/lib/seo.server';

export const headers = routeHeaders;

export async function loader(args: LoaderFunctionArgs) {
  const {params, request} = args;

  invariant(params.handle, 'Missing page handle');

  // Await the critical data required to render initial state of the page
  const criticalData = await XoBuilder.loadPageData({
    pageType: 'regular',
    args,
  });

  const {shopifyData, pageData} = criticalData;
  const {page} = shopifyData;
  const hasPage = (page && pageData.isDefault) || !pageData.isDefault;

  if (!hasPage) {
    throw new Response(null, {status: 404});
  }

  const seo = {
    ...seoPayload.page({page, url: request.url}),
    title: pageData.label,
    titleTemplate: pageData.label,
    ...criticalData.metaData,
  };

  return defer({...criticalData, seo});
}

export const meta: MetaFunction<typeof loader> = (data) => {
  return XoBuilder.pageMeta(data);
};

export default function Page() {
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


### `($locale).blogs.$blogHandle.$handle.tsx`

```diff
  import {useLoaderData, type MetaFunction} from '@remix-run/react';
import type {LoaderFunctionArgs} from '@remix-run/server-runtime';
import {defer} from '@remix-run/server-runtime';
import {XoBuilder} from '@xotiny/xb-react-elements';
import invariant from 'tiny-invariant';

import {seoPayload} from '~/lib/seo.server';

export async function loader(args: LoaderFunctionArgs) {
  const {request, params} = args;
  const {handle} = params;

  invariant(handle, 'Missing handle');

  // Await the critical data required to render initial state of the page
  const criticalData = await XoBuilder.loadPageData({
    pageType: 'article',
    args,
  });

  const {shopifyData} = criticalData;
  const {articleDetail} = shopifyData;

  if (!articleDetail) {
    throw new Response(null, {status: 404});
  }

  const seo = {
    ...seoPayload.article({article: articleDetail, url: request.url}),
    ...criticalData.metaData,
  };

  return defer({...criticalData, seo});
}

export const meta: MetaFunction<typeof loader> = (data) => {
  return XoBuilder.pageMeta(data);
};

export default function Article() {
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