# Security and Privacy Questionnaire

[Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)


>  01.  What information does this feature expose, and for what purposes?

Background Blur is supposed to enhance Privacy of the user compared to a video call without it.  Many times users are in a video call where they do not know the audience well enough. It's advisable to not accidentally share any more personal information than required which might be otherwise be exposed without any form of Background Concealment. When in doubt, it's better to blur it out and users would change the blur intensity depending on who is on the other side of the call.

> 02.  Do features in your specification expose the minimum amount of information
>      necessary to implement the intended functionality?

Yes. This specification does expose a background-blur as a capability on a video stream usually from a webcam as constrainable property.

> 03.  Do the features in your specification expose personal information,
>      personally-identifiable information (PII), or information derived from
>      either?

The information exposed by this API is not personal information or personally-identifiable information.

> 04.  How do the features in your specification deal with sensitive information?

This specification does not share additional sensitive information such as camera serial numbers or such to the Web.

That being said, this specification extends the [Media Capture and Streams](https://w3c.github.io/mediacapture-main/) specification which itself exposes video and audio streams from the user or from the user environment to Web sites with proper permissions.

> 05.  Do the features in your specification introduce state
>      that persists across browsing sessions?

No.

> 06.  Do the features in your specification expose information about the
>      underlying platform to origins?

No particular additional information is exposed about the camera hardware, like brand, etc, however we do expose if the platform has support for the constainable properties. 

> 07.  Does this specification allow an origin to send data to the underlying
>      platform?

This specification does not allow an origin access to any new sensors on a user’s device.

That being said, this specification extends the [Media Capture and Streams](https://w3c.github.io/mediacapture-main/) specification which itself allows an origin access to internal and external cameras and microphones on a user’s device per permission requests.

> 08.  Do features in this specification enable access to device sensors?

See [01]. Also note that this specification extends the [Media Capture and Streams](https://w3c.github.io/mediacapture-main/) specification which exposes video and audio streams from the user or from the user environment to Web sites per permission requests.

> 09.  Do features in this specification enable new script execution/loading
>      mechanisms?

No. 

> 10.  Do features in this specification allow an origin to access other devices?

No. See [07].

> 11.  Do features in this specification allow an origin some measure of control over
>      a user agent's native UI?

No.

> 12.  What temporary identifiers do the features in this specification create or
>      expose to the web?

[GETUSERMEDIA](https://www.w3.org/TR/mediacapture-streams/) might expose deviceID. This specification does not expose anything extra.

> 13.  How does this specification distinguish between behavior in first-party and
>      third-party contexts?

Background Blur as an extension of GetUserMedia, is exposed only to a top-level browsing secure context.

> 14.  How do the features in this specification work in the context of a browser’s
>      Private Browsing or Incognito mode?

It will work the same way, however, as soon as that session ends, the permission status will be cleaned.

> 15.  Does this specification have both "Security Considerations" and "Privacy
>      Considerations" sections?

The [Mediacapture-Extensions spec](https://w3c.github.io/mediacapture-extensions/) does not yet have a Privacy and Security section. However, this is similar to ImageCapture specification, See the [Security and
Privacy](https://w3c.github.io/mediacapture-image/#securityandprivacy) section. Background Blur and their privacy and security analysis is covered in a separate [explainer](https://github.com/riju/backgroundBlur/blob/main/explainer.md#security-considerations). This should be an addendum to the main specification's [Privacy section](https://w3c.github.io/mediacapture-main/#privacy-and-security-considerations).

> 16.  Do features in your specification enable origins to downgrade default
>      security protections?

By default, this specification uses the same permissions namely `camera` as the [mediacapture-main](https://w3c.github.io/mediacapture-main/) specification. 

> 17.  What happens when a document that uses your feature is kept alive in BFCache
>      (instead of getting destroyed) after navigation, and potentially gets reused
>      on future navigations back to the document?

> 18.  What happens when a document that uses your feature gets disconnected?

> 19.  What should this questionnaire have asked?
