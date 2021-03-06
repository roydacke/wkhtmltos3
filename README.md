### Supported tags and respective `Dockerfile` links

+ [`1.5.0`,`latest` (1.5.0/Dockerfile)](https://github.com/danlynn/wkhtmltos3/blob/1.5.0/Dockerfile)
+ [`1.4.0` (1.4.0/Dockerfile)](https://github.com/danlynn/wkhtmltos3/blob/1.4.0/Dockerfile)
+ [`1.3.0` (1.3.0/Dockerfile)](https://github.com/danlynn/wkhtmltos3/blob/1.3.0/Dockerfile)
+ [`1.2.0` (1.2.0/Dockerfile)](https://github.com/danlynn/wkhtmltos3/blob/1.2.0/Dockerfile)
+ [`1.1.0` (1.1.0/Dockerfile)](https://github.com/danlynn/wkhtmltos3/blob/1.1.0/Dockerfile)
+ [`1.0.0` (1.0.0/Dockerfile)](https://github.com/danlynn/wkhtmltos3/blob/1.0.0/Dockerfile)

This image will execute [wkhtmltoimage](https://wkhtmltopdf.org) to render an html page specified by an URL to a jpg image and then upload that image to s3.  Alternatively, this container can be launched as a service which listens to an AWS SQS queue for render messages instead of running as a cli command that renders on html page into an image then exits.

Note: The current version only supports rendering images.  Stay tuned for a future release that also supports rendering to PDF.

`wkhtmltopdf/wkhtmltoimage 0.12.4 + aws-sdk 2.48.0 + imagemagick 8:6.8.9.9-5+deb8u8 + node 6.10.2`

### How to use

The wkhtmltos3 image was originally developed to be invoked as an Amazon EC2 Container Service task that when invoked performs the process of rendering one html page into a jpg image that is stored to s3 and then exits.  However, it can be used in other contexts too.

Assuming that you have [Docker installed](https://www.docker.com/community-edition) locally, the wkhtmltos3 docker container can be invoked as follows:

```bash
$ docker run --rm -e ACCESS_KEY_ID=AKIA000NOTREALKEY000 -e SECRET_ACCESS_KEY=l2r+0000000NotRealSecretAccessKey0000000 danlynn/wkhtmltos3 -V -b my-unique-bucket -k 123/profile12345.jpg -e 1 'http://some.com/retailers/123/users/12345/profile.html'

wkhtmltos3:
  bucket:      my-unique-bucket
  key:         123/profile12345.jpg
  format:      jpg
  url:         http://some.com/retailers/123/users/12345/profile.html

  wkhtmltoimage generate ({})...
  imagemagick convert ([])...
  uploading 32.57k to s3...
  complete
```

Note, however, that the ACCESS_KEY_ID and SECRET_ACCESS_KEY environment variables can also be passed as command-line options like:

```bash
$ docker run --rm danlynn/wkhtmltos3 -V -b my-unique-bucket -k 123/profile12345.jpg -e 1 --accessKeyId=AKIA000NOTREALKEY000 --secretAccessKey=l2r+0000000NotRealSecretAccessKey0000000 'http://some.com/retailers/123/users/12345/profile.html'

wkhtmltos3:
  bucket:      my-unique-bucket
  key:         123/profile12345.jpg
  url:         http://some.com/retailers/123/users/12345/profile.html

  rendering jpg...
  wkhtmltoimage generate ({})...
  imagemagick convert ([])...
  uploading 32.57k to s3...
  complete
```

Note that if running the docker container within AWS (Amazon Web Services) as an ECS (EC2 Container Service) then the ACCESS_KEY_ID and SECRET_ACCESS_KEY can be left off because IAM will control authorization.

All logging is written to STDOUT and STDERR.  The above example used the -V option to provide verbose output.  Leaving this option off will provide output like:

Success to STDOUT (non-verbose):

```bash
wkhtmltos3: success: http://some.com/retailers/123/users/12345/profile.html => s3:my-unique-bucket:123/profile12345.jpg
```

Failure to STDERR (non-verbose):

```bash
wkhtmltos3: fail upload: http://some.com/retailers/123/users/12345/profile.html => s3:NON-EXISTENT-bucket:123/profile12345.jpg (error = NoSuchBucket: The specified bucket does not exist)
```

### Config: env vars and options

The configuration environment variables and command line options can be displayed with the `-?` or `--help` options:

```
NAME
   wkhtmltos3 - Use webkit to convert html page to image on s3

SYNOPSIS
   wkhtmltos3 [-q queueUrl] [--region] [--maxNumberOfMessages] 
              [--waitTimeSeconds] [--waitTimeSeconds] [--visibilityTimeout] 
              -b bucket [-k key]
              [--format] [--trim] [--width] [--height]
              [--accessKeyId] [--secretAccessKey]
              [-V verbose] [--wkhtmltoimage]
              [--imagemagick] [--url] [url]

DESCRIPTION
   Convert html page specified by 'url' into a jpg image and
   upload it to amazon s3 into the specified 'bucket' and
   'key'. Can be run as either a single invocation that uses the
   command-line options to identify 'url', 'key', etc. to render
   an html page to an image on s3 -OR- can be launched as a service
   that listens for messages to be posted to an aws SQS queue. If
   '--queueUrl' is specified then it will launch as a service.

   -q, --queueUrl=queueUrl
           url of an aws SQS queue to listen for messages
   --region=region_name
           aws availability zone of SQS queue
   --maxNumberOfMessages=number
           max number of messages to retrieve and process at a time
           (default 5)
   --waitTimeSeconds=number
           Amount of time to wait for messages before giving up. 
           Values > 0 invoke long polling for efficiency.
           (default 10 seconds)
   --visibilityTimeout=number
           Amount of time before SQS queue will make a message 
           available to be received again (in case error occurred
           and the message was not processed then deleted)
           (default 15 seconds)
   -b, --bucket=bucket_name
           amazon s3 bucket destination
   -k, --key=filename
           key in amazon s3 bucket
   --format=format
           image file format (default is jpg)
   --trim
           use imagemagick's trim command to automatically crop
           whitespace from images since html pages always default
           to 1024 wide and the height usually has some padding 
           too
           see: http://www.imagemagick.org/Usage/crop/#trim
   --width=pixels
           explicitly set the width for wkhtmltoimage rendering
   --height=pixels
           explicitly set the height for wkhtmltoimage rendering
   --accessKeyId=ACCESS_KEY_ID
           Amazon accessKeyId that has access to bucket - if not
           provided then 'ACCESS_KEY_ID' env var will be used.
           If running within the aws environment (ec2, etc)
           then this value is optional.
   --secretAccessKey=SECRET_ACCESS_KEY
           Amazon secretAccessKey that has access to bucket - if
           not provided then 'SECRET_ACCESS_KEY' env var will be
           used. If running within the aws environment (ec2, etc)
           then this value is optional.
   --wkhtmltoimage=json_array
           options (in json array format) to be passed through directly 
           to the wkhtmltoimage cli tool as command line options. 
           (eg: --wkhtmltoimage='["--zoom", 2.0]'). These options will 
           merge into and override any of the regular options 
           (like --width=400, --format=png, etc).
           see: https://wkhtmltopdf.org/usage/wkhtmltopdf.txt
   --imagemagick=json_array
           options (in json array format) to be passed through directly
           to the imagemagick node module. This is a highly flexible
           way to perform additional image manipulation on the rendered
           html page. (eg: --imagemagick='["-trim","-colorspace","Gray",
           "-edge",1,"-negate"]')
   --url=url
           optionally explicitly identify the url instead of just
           tacking it on the end of the command-line options
   -V, --verbose
           provide verbose logging
   -P, --profile
           log execution timing info at end of run
   -?, --help
           display this help
```

### Trimming and sizing jpg image

wkhtmltopdf/wkhtmltopdf is great at rendering html pages to jpg images and PDF files.  However, since the source is an html page, it makes certain assumptions about the size of the page.  

If the source of the page is something small like a coupon, you may be disappointed that the default rendering produces a 1024px wide image with a lot of padding on the right and probably some on the bottom.

![non-trimmed coupon](https://github.com/danlynn/wkhtmltos3/raw/master/assets/no_trim.jpg "Non-trimmed Coupon")

In order to correct for this common problem, the `--trim` option has been added.  The `--trim` option will use the [`-trim`](http://www.imagemagick.org/Usage/crop/#trim) feature of imagemagick to automagically crop extra whitespace from your rendered jpg image.

![trimmed coupon](https://github.com/danlynn/wkhtmltos3/raw/master/assets/trim.jpg "Trimmed Coupon")

However, if the automatic nature of this feature doesn't work for the types of html pages being rendered then you can explicitly specify `--width=<pixels>` and/or `--height=<pixels>` to set the page size used by wkhtmltoimage/wkhtmltopdf when rendering.

### Pass-through config options

The `--wkhtmltoimage` and `--imagemagick` options allow you to pass through options directly to the wkhtmltoimage binary and imagemagick node module. This exposes some really useful capabilities.

#### --wkhtmltoimage options

For example, for wkhtmltoimage, you can specify that the image should be zoomed by 200% in order to produce retina resolution images.

```bash
$ docker run --rm -e ACCESS_KEY_ID=AKIA000NOTREALKEY000 -e SECRET_ACCESS_KEY=l2r+0000000NotRealSecretAccessKey0000000 danlynn/wkhtmltos3 -V -b my-unique-bucket -k 123/profile12345.jpg -e 1 --wkhtmltoimage='["zoom": 2.0]' 'http://some.com/retailers/123/users/12345/profile.html'

wkhtmltos3:
  bucket:      my-unique-bucket
  key:         123/profile12345.jpg
  format:      jpg
  url:         http://some.com/retailers/123/users/12345/profile.html

  wkhtmltoimage generate (["zoom": 2.0])...
  imagemagick convert ([])...
  uploading 32.57k to s3...
  complete
```

You can see all of the wkhtmltopdf/wkhtmltoimage options on the wkhtmltopdf website.

options reference: [https://wkhtmltopdf.org/usage/wkhtmltopdf.txt](https://wkhtmltopdf.org/usage/wkhtmltopdf.txt)

node module: [https://www.npmjs.com/package/wkhtmltoimage](https://www.npmjs.com/package/wkhtmltoimage)

#### --imagemagic options

Similarly, options can be passed directly through to the imagemagic node module via the `--imagemagic` option as a json array string. In this case the option names are the same as appears in the reference documentation (no camel-case conversion, thankfully).

For example, an edge filter can be applied to the image rendered from the html page via:

```bash
$ docker run --rm -e ACCESS_KEY_ID=AKIA000NOTREALKEY000 -e SECRET_ACCESS_KEY=l2r+0000000NotRealSecretAccessKey0000000 danlynn/wkhtmltos3 -V -b my-unique-bucket -k 123/14106.jpg -e 1 --trim --imagemagick='["-colorspace","Gray","-edge",1,"-negate"]' 'http://some.com/retailers/123/coupons/14106'

wkhtmltos3:
  bucket:      my-unique-bucket
  key:         123/14106.jpg
  format:      jpg
  url:         http://some.com/retailers/123/coupons/14106

  wkhtmltoimage generate ({})...
  imagemagick convert (["-trim","-colorspace","Gray","-edge",1,"-negate"])...
  uploading 32.57k to s3...
  complete
```

Producing the following image:

![edge filtered coupon](https://github.com/danlynn/wkhtmltos3/raw/master/assets/edge.jpg "Edge Filtered Coupon")

Note that the `--trim` option to wkhtmltos3 was simply merged into the other imagemagick options as `"-trim"`.

imagemagick reference: [http://www.imagemagick.org/Usage/](http://www.imagemagick.org/Usage/)

node module: [https://www.npmjs.com/package/imagemagick](https://www.npmjs.com/package/imagemagick)

### Font Handling

The docker container has only the default fonts available on the Debian 8 base image.  These fonts can be displayed by launching the container into bash and using the `fc-list` command:

```bash
root@684fc69c5877:/myapp$ fc-list

/usr/share/fonts/truetype/dejavu/DejaVuSerif-Bold.ttf: DejaVu Serif:style=Bold
/usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf: DejaVu Sans Mono:style=Book
/usr/share/fonts/X11/Type1/c0649bt_.pfb: Bitstream Charter:style=Italic
/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf: DejaVu Sans:style=Book
/usr/share/fonts/X11/Type1/c0419bt_.pfb: Courier 10 Pitch:style=Regular
/usr/share/fonts/X11/Type1/c0633bt_.pfb: Bitstream Charter:style=Bold Italic
/usr/share/fonts/X11/Type1/c0648bt_.pfb: Bitstream Charter:style=Regular
/usr/share/fonts/X11/Type1/c0611bt_.pfb: Courier 10 Pitch:style=Bold Italic
/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf: DejaVu Sans:style=Bold
/usr/share/fonts/truetype/dejavu/DejaVuSansMono-Bold.ttf: DejaVu Sans Mono:style=Bold
/usr/share/fonts/X11/Type1/c0632bt_.pfb: Bitstream Charter:style=Bold
/usr/share/fonts/X11/Type1/c0582bt_.pfb: Courier 10 Pitch:style=Italic
/usr/share/fonts/X11/Type1/c0583bt_.pfb: Courier 10 Pitch:style=Bold
/usr/share/fonts/truetype/dejavu/DejaVuSerif.ttf: DejaVu Serif:style=Book
```

This is a pretty minimal list.  However, wkhtmltopdf does fully support web fonts via webkit.  Thus, you can make any other fonts that you need available via @font-face css rules like:

```css
@font-face {
	font-family: 'core-icons';
	src:url('core-icons.eot');
	src:url('core-icons.eot') format('embedded-opentype'),
		url('core-icons.woff') format('woff'),
		url('core-icons.ttf') format('truetype'),
		url('core-icons.svg') format('svg');
	font-weight: normal;
	font-style: normal;
}
```

...which can use web fonts from google or fonts hosted on your own web servers.

### Running wkhtmltos3 as a service which listens to an AWS SQS queue

If you start the docker container passing the optional `--queueUrl=<queueUrl>` and `--region=<region>` options then wkhtmltos3 will run as a service that runs continuously listening for render messages on the AWS SQS (Simple Queue Service).  Note that the AWS SQS must be setup such that it is backed by Redis (not Memcache).

```bash
$ node src/wkhtmltos3.js -V --queueUrl https://sqs.us-east-1.amazonaws.com/018867421119/dynamic-email-render --region=us-east-1 -b webstop-dynamic-email -e 1 --trim -P
```

Any options that are passed on the command line when launching as a service will act as defaults which will be overridden by options provided in the render messages.

The format of the render messages should be as a JSON object where the attribute names are the long names of the wkhtmltos3 command line options and the values are as defined in the help `-?`.

Example JSON render messages:

```json
{"url": "http://api.grocerywebsite.com/retailers/767/coupons/28967/dynamic", "key": "test/queue1.jpg", "trim": true, "imagemagick": ["-trim","-colorspace","Gray", "-edge",1,"-negate"], "wkhtmltoimage": ["--zoom", 2.0]}
{"url": "http://api.grocerywebsite.com/retailers/767/coupons/28967/dynamic", "key": "test/queue2.jpg", "trim": true, "wkhtmltoimage": ["--zoom", 2.0]}
{"url": "http://api.grocerywebsite.com/retailers/767/coupons/28967/dynamic", "key": "test/queue3.jpg", "trim": true, "imagemagick": ["-trim","-colorspace","Gray", "-edge",1,"-negate"]}
{"url": "http://api.grocerywebsite.com/retailers/767/coupons/28967/dynamic", "key": "test/queue4.jpg", "trim": true}
{"url": "http://api.grocerywebsite.com/retailers/767/coupons/28967/dynamic", "key": "test/queue5.jpg"}
```

Some command line options are not valid and will be ignored if they appear in the render messages.  The ignored options are: `--queueUrl`, `--region`, `--maxNumberOfMessages`, `--waitTimeSeconds`, `--visibilityTimeout`, `--accessKeyId`, `--secretAccessKey`

You can try out different render messages manually in the SQS Management Console by selecting your queue and then selecting 'Send a Message' from the 'Queue Actions' drop-down.

### How to develop/customize wkhtmltos3

Check out the project from github at: [https://github.com/danlynn/wkhtmltos3](https://github.com/danlynn/wkhtmltos3)

Make changes to the Dockerfile and build with:

```bash
$ docker build -t danlynn/wkhtmltos3:1.1.0 .
```

...replacing the tag (-t) value as needed.

Launch the image and make interactive changes to the wkhtmltos3.js by mounting the current project directory in the container and opening a bash prompt via:

```bash
$ docker run --rm -it -v $(pwd):/myapp --entrypoint=/bin/bash -e ACCESS_KEY_ID= AKIA000NOTREALKEY000 -e SECRET_ACCESS_KEY= l2r+0000000NotRealSecretAccessKey0000000 danlynn/wkhtmltos3:1.1.0
```

Then from the bash prompt in the container, run the script with your modifications via:

```bash
root@684fc69c5877:/myapp$ node wkhtmltos3.js -V -b my-unique-bucket -k 123/profile12345.jpg -e 1 'http://some.com/retailers/123/users/12345/profile.html'

wkhtmltos3:
  bucket:      my-unique-bucket
  key:         123/profile12345.jpg
  format:      jpg
  url:         http://some.com/retailers/123/users/12345/profile.html

  wkhtmltoimage generate ({})...
  imagemagick convert ([])...
  uploading 32.57k to s3...
  complete
```
