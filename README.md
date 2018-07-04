# Vue Storefront integration easy as 1-2-3

Vue Storefront is platform agnostic which means it can be connected to virtually any CMS. Currently we do officially support:
- [Magento2]() via `mage2vuestorefront` + `vue-storefront-api` data bridge,
- [Magento1](https://github.com/DivanteLtd/magento1-vsbridge) via `magento1-vsbrdige` data bridge.
- Limited: [Pimcore](https://github.com/DivanteLtd/pimcore2vuestorefront) via `pimcore2vuestorefront` data bridge,

In this repository You can find materials that will let You integrate any other 3rd party platform or bespoke e-Commerce backend with the Vue Storefront.

## Three steps of integration

- **Layer A** Vue Storefront uses Elastic Search as backend for all catalog operations. It stores products, categories, attributes and tax-rules in the ES and fetch data from it using [`vue-storefront-api`](https://github.com/DivanteLtd/vue-storefront-api) project.

- **Layer B** The second layer are so called **dynamic calls** that are used to synchronize shopping carts, promotion rules, user accounts etc.

However, `vue-storefront-api` was initially meant to support all kind of requests (and it's supporting them for Magento2 currently) - we found that's much easier to create a **dedicated api** which provides the same data formats like `vue-storefront-api` around Your platform of choice, than adding very customized data adapters in the `vue-storefront-api` itself.

That means that creating the Vue Storefront integration basically is divided into 3 main steps:

1. [You exposes the API around Your platform]() of choice regarding our spec. In fact it's a `vue-storefront-api` enpodints specification. Having such an API - Vue Storefront will be able to speak directly with Your platform as a backend **Layer B**.

2. [You need to use `node-app` application]() to import the data from the freshly created API endpoints to the Elastic Search **Layer A**.


3. [Then You just need to point `vue-storefront`]() application to newly created endpoints.



## Step 1: How to expose the API around Your platform

You may take a look [how we did it for the Magento1](https://github.com/DivanteLtd/magento1-vsbridge/tree/master/magento1-module/app). It was a bunch of additional Magento1 api methods we created to fullfill the [API specification](https://github.com/DivanteLtd/vue-storefront-integration-boilerplate/tree/master/1.%20Expose%20the%20API%20endpoints%20required%20by%20VS). You can create a separate nodejs/php/rails/whatever app if Your platform doesn't exposes the required API and just fullfill the spec.

## Step 2: How to use node-app

Then, having the API in place You're to fo the **Layer A** integration. Which means - to fill the ElasticSearch with products, categories, attributes and taxrules data.

You can use our [boilerplate node-app]() which is compilant with the data formats specified above.
This is a consumer application that's responsible for synchronizing the Magento1 data with the ElasticSearch instance.

This tool required ElasticSearch instance up and running. The simplest way to have one is to install [vue-storefront](https://github.com/DivanteLtd/vue-storefront) and [vue-storefront-api](https://github.com/DivanteLtd/vue-storefront-api) and run `docker-compose up` inside `vue-storefront-api` installation as the project [contains Docker file](https://github.com/DivanteLtd/vue-storefront-api/blob/master/docker-compose.yml) for Vue Storefront.

Then you need to modify the configs:

```
cd node-app/src
cp config.example.json config.json
nano config.json
```

In the config file please setup the following variables:
- `auth` section to setup user login and password - these values will be used to generate the JWT token used to authorize the requests against Your API - 
- ['endpoint'](https://github.com/DivanteLtd/magento1-vsbridge/blob/5d4b9285c2dd2a20900e6075f50ebc2d7802499e/node-app/config.example.json#L14) should match your Magento 1.9 URL,
- [`elasticsearch.indexName`](https://github.com/DivanteLtd/magento1-vsbridge/blob/5d4b9285c2dd2a20900e6075f50ebc2d7802499e/node-app/config.example.json#L4) should be set to Your ElasticSearch index which then will be connected to the Vue Storefront. It can be fresh / non-existient index as well (will be created then). For example You may have: `vue_storefront_mangento1`


## Step 3: How to configure vue-storefront