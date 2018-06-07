# Image Optimization (IO) Guide

This guide will show how enable Fastly Image Optimization from the Magento admin and discuss configurable options.

## Prerequisite

* You have to have IO feature enabled for your Fastly service. IO is a contracted add-on.
[To do: add that customers should upgrade to plugin version 1.2.5.?]
 
## Enablement
 
Once you're ready, go to: 

```
Magento admin > Stores > Configuration > Advanced > System > Full Page Cache > Fastly Configuration
```

Click on the **Image Optimization** tab. 

You will be presented with a dialog such as this

<img alt="IO Main Screen" title="IO Main Screen" src="../images/guides/image-optimization/io_main_screen.png" width="800px"/>

If instead you receive this message

<img alt="IO Not Enabled" title="IO Not Enabled" src="../images/guides/image-optimization/io_not_enabled.png" width="600px"/>

it means you do not have the IO feature enabled and need to talk to our sales team.

There are four main categories

* Fastly IO Snippet Upload
* Default IO config options
* Deep image optimization
* Adaptive pixel ratios

## Fastly IO Snippet Upload

IO Snippet Upload is required in order to enable IO on your images. It inserts a VCL snippet in your Fastly service
which instructs Fastly to process all images through our Image optimizers. This snippet will use default IO config
options to process images. Things to note

* It will not do any other transformations e.g. cropping, rotations etc.
* It will not do any image format changes except for browsers supporting WebP (Chrome etc.)
* PNG images will not see much improvement since they are lossless and it's generally hard to compress them further

For the curious clicking Enable will

* Upload [https://github.com/fastly/fastly-magento2/blob/master/etc/vcl_snippets_image_optimizations/recv.vcl](https://github.com/fastly/fastly-magento2/blob/master/etc/vcl_snippets_image_optimizations/recv.vcl) snippet
* Configure default IO config options to use WebP for browsers supporting it (auto=webp) and optionally set the JPEG and WebP image
quality levels to the value set in Magento. If the option is not set we'll default to quality 85 for lossy images.

## Default IO config options

Default IO config options allow you to tweak default settings that Image optimizers use to optimize your images

<img alt="IO Default Config Options" title="IO Default Config Options" src="../images/guides/image-optimization/io_default_config_dialog.png" width="800px"/>

Things you may commonly change are WebP and JPEG quality levels for lossy formats or whether to serve Progressive or Baseline JPEGs.

## Deep image optimization

Deep image optimization is off by default. Enabling it will turn off built-in Magento resizing and offload it
completely to Fastly IO. It only applies to *product* images. CMS images are not resized.

Please note that deep image optimization will add background color definition to every image as defined in your
theme. This will result in WebP images switching from WebP lossless to WebP lossy. One of the major differences
between lossless and lossy is that it drops the alpha channel from PNG images. This will result in much smaller
images however may result in images with transparencies looking odd on product and campaign pages that use a
different background.

As an example an image from the Luma theme that originally looked like this

```
 <img class="product-image-photo"
    src="https://mymagentosite/pub/media/catalog/product/cache/f073062f50e48eb0f0998593e568d857/m/b/mb02-gray-0.jpg"
    width="240"
    height="300"
  alt="Fusion Backpack"/>
```
will be rewritten as this

```
 <img class="product-image-photo"
    src="https://mymagentosite/pub/media/catalog/product/m/b/mb02-gray-0.jpg?width=240&height=300&quality=80&bg-color=255,255,255&fit=bounds"
    width="240"
    height="300"
  alt="Fusion Backpack"/>
```

## Adaptive pixel ratios

This functionality enables delivering a fixed-width image that can adapt to varying `device-pixel-ratios`.
It will add srcsets to product images. 
Learn about `srcset` [browser support](https://caniuse.com/#feat=srcset) and [specification](https://html.spec.whatwg.org/multipage/embedded-content.html#attr-img-srcset). 
For example the product image definition will be rewritten as follows

```
<img class="product-image-photo"
  srcset="https://mymagentosite/pub/media/catalog/product/m/b/mb02-gray-0.jpg?width=240&height=300&quality=80&bg-color=255,255,255&fit=bounds&dpr=2 2x,
  https://mymagentosite/pub/media/catalog/product/m/b/mb02-gray-0.jpg?width=240&height=300&quality=80&bg-color=255,255,255&fit=bounds&dpr=3 3x"
  src="https://mymagentosite/pub/media/catalog/product/m/b/mb02-gray-0.jpg?width=240&height=300&quality=80&bg-color=255,255,255&fit=bounds"
    width="240"
    height="300"
  alt="Fusion Backpack"/>
```
