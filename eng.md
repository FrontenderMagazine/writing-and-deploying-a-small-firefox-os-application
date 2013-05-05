# Writing and deploying a small Firefox OS application

For the last week I’ve been using a [Geeksphone Keon][1] as my only phone. There’s
been no cheating here, I don’t have a backup Android phone and I’ve not taken
to carrying around a tablet everywhere I go (though its use has increased at
home slightly…) On the whole, the experience has been positive. Considering
how entrenched I was in Android applications and Google services, it’s been
surprisingly easy to make the switch. I would recommend anyone getting the
Geeksphones to [build their own OS images][2] though, the shipped images are pretty
poor.

Among the many things I missed (Spotify is number 1 in that list btw), I could
have done with a countdown timer. Contrary to what the interfaces of most
Android timer apps would have you believe, it’s not rocket-science to write a
usable timer, so I figured this would be a decent entry-point into writing
mobile web applications. For the most part, this would just be your average
web-page, but I did want it to feel ‘native’, so I started looking at the
[new building blocks site][3] that documents the FirefoxOS shared resources. I had
elaborate plans for tabs and headers and such, but turns out, all I really
needed was the button style. The site doesn’t make hugely clear that you’ll
actually need to check out the shared resources yourself, which can be found
on [GitHub][4].

Writing the app was easy, except perhaps for getting things to align
vertically (for which I used the nested div/”display: table-cell; vertical-
alignment: middle;” trick), but it was a bit harder when I wanted to use some
of the new APIs. In particular, I wanted the timer to continue to work when
the app is closed, and I wanted it to alert you only when you aren’t looking
at it. This required use of the [Alarm API][5], the [Notifications API][6] and
the [Page Visibility API][7].

The page visibility API was pretty self-explanatory, and I had no issues using
it. I use this to know when the app is put into the background (which,
handily, always happens before closing it. I think). When the page gets
hidden, I use the Alarm API to set an alarm for when the current timer is due
to elapse to wake up the application. I found this particularly hard to use as
the documentation is very poor (though it turns out the code you need is quite
short). Finally, I use the Notifications API to spawn a notification if the
app isn’t visible when the timer elapses. Notifications were reasonably easy
to use, but I’ve yet to figure out how to map clicking on a notification to
raising my application – I don’t really know what I’m doing wrong here, any
help is appreciated!

The last hurdle was deploying to an actual device, as opposed to the
[simulator][8]. Apparently the simulator has a deploy-to-device feature, but this
wasn’t appearing for me and it would mean having to fire up my Linux VM (I
have my reasons) anyway, as there are currently no Windows drivers for the
Geeksphone devices available. I obviously don’t want to submit this to the
Firefox marketplace yet, as I’ve barely tested it. I have my own VPS, so
ideally I could just upload the app to a directory, add a meta tag in the
header and try it out on the device, but unfortunately it isn’t as easy as
that.

Getting it to work well as a web-page is a good first step, and to do that
you’ll want to add a [meta viewport tag][9]. Getting the app to install itself from
that page was easy to do, but difficult to find out about. I think the process
for this is harder than it needs to be and quite poorly documented, but
basically, you want this in your app:

    if (navigator.mozApps) {
      var request = navigator.mozApps.getSelf();
      request.onsuccess = function() {
        if (!this.result) {
          request = navigator.mozApps.install(location.protocol + "//" + location.host + location.pathname + "manifest.webapp");
          request.onerror = function() {
            console.log("Install failed: " + this.error.name);
          };
        }
      };
    }

And you want all paths in your [manifest][10] and [appcache][11] manifest to be absolute
(you can assume the host, but you can’t have paths relative to the directory
the files are in). This last part makes deployment very awkward, assuming you
don’t want to have all of your app assets in the root directory of your server
and you don’t want to setup vhosts for every app. You also need to make sure
your server has the [webapp mimetype][12] setup. Mozilla has a great
[online app validation tool][13] that can help you debug problems in this process.

![Screenshot][Our application]
And we’re done! (Ctrl+Shift+M to toggle responsive design mode in Firefox)

Visiting the page will offer to install the app for you on a device that
supports app installation (i.e. a Firefox OS device). Not bad for a night’s
work! Feel free to laugh at my n00b source and tell me how terrible it is in
the comments

[1]: http://www.geeksphone.com/#feat
[2]: https://developer.mozilla.org/en-US/docs/Mozilla/Firefox_OS/Building
[3]: http://buildingfirefoxos.com/
[4]: https://github.com/mozilla-b2g/gaia/tree/master/shared
[5]: https://developer.mozilla.org/en-US/docs/WebAPI/Alarm
[6]: https://developer.mozilla.org/en-US/docs/DOM/Displaying_notifications
[7]: https://developer.mozilla.org/en-US/docs/DOM/Using_the_Page_Visibility_API
[8]: https://addons.mozilla.org/en-US/firefox/addon/firefox-os-simulator/
[9]: https://developer.mozilla.org/en-US/docs/Mobile/Viewport_meta_tag
[10]: https://developer.mozilla.org/en-US/docs/Apps/Manifest
[11]: https://developer.mozilla.org/en-US/docs/HTML/Using_the_application_cache
[12]: https://developer.mozilla.org/en-US/docs/Apps/Manifest#Serving_manifests
[13]: https://marketplace.firefox.com/developers/validator

[Our application]: img/timer.png "Our application"

