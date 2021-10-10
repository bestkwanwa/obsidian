# Next获取数据的方法
`getStaticProps, getStaticPaths, getInitialProps and getServersideProps`
## What is build-time?
`build-time`发生在`next build`。`runtime`是应用在生产环境中运行（`next start`）。
## getInitialProps
`getInitialProps`在`客户端`和`服务端`运行时都会执行。
## getServersideProps
`getServersideProps`只在`服务端`运行时执行。
## getStaticProps
`getStaticProps`只在`build-time`时运行。
## getStaticPaths
`getStaticPaths`只在`build-time`时运行。