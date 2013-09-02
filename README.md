activitystreams-unofficial ![ActivityStreams](https://raw.github.com/snarfed/activitystreams-unofficial/master/static/logo_small.png)

About
===

This is a library and REST API that converts Facebook, Twitter, and Instagram
data to [ActivityStreams](http://activitystrea.ms/) format. You can try it out
with these interactive demos:

http://facebook-activitystreams.appspot.com/  
http://twitter-activitystreams.appspot.com/  
http://instagram-activitystreams.appspot.com/

It's part of a suite of projects that implement the
[OStatus](http://ostatus.org/) federation protocols for the major social
networks. The other projects include
[portablecontacts-](https://github.com/snarfed/portablecontacts-unofficial),
[salmon-](https://github.com/snarfed/salmon-unofficial),
[webfinger-](https://github.com/snarfed/webfinger-unofficial), and
[ostatus-unofficial](https://github.com/snarfed/ostatus-unofficial).

License: This project is placed in the public domain.


Using
===

The library and REST API are both based on the
[OpenSocial Activity Streams service](http://opensocial-resources.googlecode.com/svn/spec/2.0.1/Social-API-Server.xml#ActivityStreams-Service).

Let's start with an example. This method call in the library:

    from activitystreams_unofficial import twitter
    ...
    tw = twitter.Twitter(handler)
    tw.get_activities(group_id='@friends')

is equivalent to this `HTTP GET` request:

`https://twitter-activitystreams.appspot.com/@me/@friends/@app/?access_token_key=KEY&access_token_secret=SECRET`

They return the authenticated user's Twitter stream, ie tweets from the people they
follow. Here's the JSON output:

    {
      "itemsPerPage": 10,
      "startIndex": 0,
      "totalResults": 12
      "items": [{
          "verb": "post",
          "id": "tag:twitter.com,2013:374272979578150912"
          "url": "http://twitter.com/evanpro/status/374272979578150912",
          "content": "Getting stuff for barbecue tomorrow. No ribs left! Got some nice tenderloin though. (@ Metro Plus Famille Lemay) http://t.co/b2PLgiLJwP",
          "actor": {
          "username": "evanpro",
            "displayName": "Evan Prodromou",
            "description": "Prospector.",
            "url": "http://twitter.com/evanpro",
          },
          "object": {
            "tags": [{
                "url": "http://4sq.com/1cw5vf6",
                "startIndex": 113,
                "length": 22,
                "objectType": "article"
              }, ...],
          },
        }, ...]
      ...
    }

The request parameters are the same for both, all optional: `user_id` is a
source-specific id or `@me` for the authenticated user. `group_id` may be
`@all`, `@friends` (currently identical to `@all`), or `@self`. `app_id` is
currently ignored; best practice is to use `@app` as a placeholder.

Paging is supported via the `startIndex` and `count` parameters. They're self
explanatory, and described in detail in the
[OpenSearch spec](http://www.opensearch.org/Specifications/OpenSearch/1.1#The_.22count.22_parameter)
and
[OpenSocial spec](http://opensocial-resources.googlecode.com/svn/spec/2.0.1/Social-API-Server.xml#ActivityStreams-Service).

Output data is
[JSON Activity Streams 1.0](http://activitystrea.ms/specs/json/1.0/) objects
wrapped in the
[OpenSocial envelope]http://opensocial-resources.googlecode.com/svn/spec/2.0.1/Social-API-Server.xml#ActivityStreams-Service).

Most requests will need an OAuth access token from the source provider. Here are
their authentication docs:
[Facebook](https://developers.facebook.com/docs/facebook-login/access-tokens/),
[Twitter](https://dev.twitter.com/docs/auth/3-legged-authorization),
[Instagram](http://instagram.com/developer/authentication/).

If you get an access token and pass it along, it will be used to sign and
authorize the underlying requests to the sources providers. See the demos on the
REST API [endpoints above](#About) for examples.


Using the REST API
===

The [endpoints above](#About) all serve the
[OpenSocial Activity Streams REST API](http://opensocial-resources.googlecode.com/svn/spec/2.0.1/Social-API-Server.xml#ActivityStreams-Service).
Request paths are of the form:

`/USER_ID/GROUP_ID/APP_ID/ACTIVITY_ID?startIndex=...&count=...&format=FORMAT&access_token=...`

All query parameters are optional.
`FORMAT` may be `json` (the default), `xml`, or `atom`, both of which return
[Atom](http://www.intertwingly.net/wiki/pie/FrontPage).
The rest of the path elements and query params are [described above](#Using).

Errors are returned with the appropriate HTTP response code, e.g. 403 for
Unauthorized, with details in the response body.

To use the REST API in an existing ActivityStreams client, you'll need to
hard-code exceptions for the domains you want to use e.g. `facebook.com`, and
redirect HTTP requests to the corresponding [endpoint above](#About).


Using the library
===

See the [example above](#Using) for a quick start guide.

Clone or download this repo into a directory named `activitystreams_unofficial`
(note the underscore instead of dash). Each source works the same way. Import
the module for the source you want to use, and instantiate its class by passing
the HTTP handler object. It should have a `request` attribute for the current
HTTP request.

The useful methods are `get_activities()` and `get_actor()`, which returns the
current authenticated user (if any). See the
[individual method docstrings](https://github.com/snarfed/activitystreams-unofficial/blob/master/source.py)
for details. All return values are Python dicts of decoded ActivityStreams JSON.

The `activitystreams.render_html()` function is also useful for rendering an
ActivityStreams object as nicely formatted HTML.


History
===

[Cliqset's FeedProxy](http://www.readwriteweb.com/archives/cliqset_activity_streams_api.php)
used to do this same kind of format translation, but unfortunately it and
Cliqset died.

Facebook
[used to](https://developers.facebook.com/blog/post/225/)
[officially](https://developers.facebook.com/blog/post/2009/08/05/streamlining-the-open-stream-apis/)
[support](https://groups.google.com/forum/#!topic/activity-streams/-b0LmeUExXY)
ctivityStreams, but that's also dead.


Future work
===

The REST APIs are currently much more usable than the library. We need to make
the library easier to use. Most of the hard work is already done; here's what remains.

  * Allow passing OAuth tokens as keyword args.
  * Expose the initial OAuth permission flow. The hard work is already done, we
    just need to let users trigger it programmatically.
  * Expose the `format` arg and let users request
    [Atom](http://www.intertwingly.net/wiki/pie/FrontPage) output.
  * Clean up and document `activitystreams.render_html()`.

We'd also love to add more sites! Off the top of my head,
[YouTube](http://youtu.be/), [Tumblr](http://tumblr.com/),
[WordPress.com](http://wordpress.com/),
[Sina Weibo](http://en.wikipedia.org/wiki/Sina_Weibo),
[Qzone](http://en.wikipedia.org/wiki/Qzone), and
[RenRen](http://en.wikipedia.org/wiki/Renren) would be good candidates. If
you're looking to get started, implementing a new site is a good place to start.
It's pretty self contained and the existing sites are good examples to follow,
but it's a decent amount of work, so you'll be familiar with the whole project
by the end.


Development
===

Pull requests are welcome! Feel free to [ping me](http://snarfed.org/about) with
any questions.

All dependencies are included as git submodules. Be sure to run `git submodule
init` after cloning this repo.

[This ActivityStreams validator](http://activitystreamstester.appspot.com/) is
useful for manual testing.

You can run the unit tests with `./alltests.py`. They depend on the
[App Engine SDK](https://developers.google.com/appengine/downloads) and
[mox](http://code.google.com/p/pymox/), both of which you'll need to install
yourself.

Note the `app.yaml.*` files, one for each App Engine app id. To work on or deploy
a specific app id, `symlink app.yaml` to its `app.yaml.xxx` file. Likewise, if you
add a new site, you'll need to add a corresponding `app.yaml.xxx` file.

To deploy:
    rm -f app.yaml && ln -s app.yaml.twitter app.yaml && \
      ~/google_appengine/appcfg.py --oauth2 update . && \
    rm -f app.yaml && ln -s app.yaml.facebook app.yaml && \
      ~/google_appengine/appcfg.py --oauth2 update . && \
    rm -f app.yaml && ln -s app.yaml.instagram app.yaml && \
      ~/google_appengine/appcfg.py --oauth2 update .
