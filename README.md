## 🔄 Cập Nhật Cấu Trúc HTML với `root.tsx`

```diff
+ <XoBuilder.Root>
  <html lang={locale.language}>
+   <XoBuilder.Root.Head jsUrls={[wcJs]}>
      <meta charSet="utf-8" />
      <meta name="viewport" content="width=device-width,initial-scale=1" />
      <meta
        name="msvalidate.01"
        content="A352E6A0AF9A652267361BBB572B8468"
      />
      <link rel="stylesheet" href={styles}></link>
      <Meta />
      <Links />
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