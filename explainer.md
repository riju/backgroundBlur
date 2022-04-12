# Background Blur

## Authors:

- Rijubrata Bhaumik
- Eero Häkkinen

## Participate
- github.com/riju/backgroundBlur/issues/

## Introduction

Background Blur has become one of the most used features on Video conferencing web apps like [Teams](https://www.microsoft.com/en-us/microsoft-teams/virtual-meeting-backgrounds), [Meet](https://workspaceupdates.googleblog.com/2020/09/blur-your-background-in-google-meet.html), [Zoom](https://support.zoom.us/hc/en-us/articles/360061468611-Using-blurred-background-), [Webex](https://help.webex.com/en-us/article/0p4gb1/Webex-App-%7C-Use-a-virtual-or-blurred-background-in-calls-and-meetings), etc. We want to achieve the goal of giving all the Web apps similar levers as their native counterparts, leveraging the same platform APIs and to delight users without completely relying on ML frameworks like TensorFlow.js, [Mediapipe](https://ai.googleblog.com/2020/10/background-features-in-google-meet.html), etc, or cloud based solutions.

## Use Cases

A vast majority of communication these days happens on our client devices. During video meetings, participants are usually aware of how they look and what their environment (usually their home) is revealing to the audience. Most folks, specially ones without a dedicated office space would be inclined to hide messy rooms with pets and kids. Video meetings like face to face meetings are important for non-verbal communication but participants would rather focus on the important subject by removing the distractions in the background and prevent any accidental snafus. [Microsoft says](https://www.microsoft.com/en-ww/microsoft-365/business-insights-ideas/resources/how-custom-backgrounds-keep-the-focus-on-you) in a 38 minute conference call, 13 minutes are wasted dealing with distractions and interruptions. Background Blur goes a long way to cutting down those disruptions. [Zoom says](https://support.zoom.us/hc/en-us/articles/360061468611-Using-blurred-background)- "_When a custom virtual background is unavailable or not suiting your needs, but you still want to maintain some privacy with regards to your surroundings, the blur background option can be a great alternative. This option simply blurs the background of your video, obscuring exactly who or what is behind you. It's great for hiding a cluttered dorm room, taking a meeting in a coffee shop, or just keeping things professional._" . In fact, NCSC (National Cyber Security Centre UK) [suggests using background Blur or a background image](https://www.ncsc.gov.uk/guidance/video-conferencing-services-security-guidance-organisations) for staff meetings to add a degree of personal privacy.

On the Web, due to a lack of a stadardized JS API for Background Blur and widespread demand, developers have no options but to use ML frameworks like Tensorflow.js and other WASM libraries to satify their customers. This Background Blur API gives developers a **choice** to use the native platform's API. This would ensure conformance to the corresponding native apps.


## Goals

* Ideally, Background Blur API should work with APIs like Face Detection. Face detection would help to either provide a mask (countour) or at least the bounding box co-ordinates which might help Background Blur to compute faster. In many cases platforms will do an in-stream correction and explicit face detection might not be needed at the browser level. In that case the detectedFaces options can be nullable.

* Background Blur API should have options similar to what consumers demand, things possible on ML frameworks but not yet exposed by present day Platform APIs like Blur Level. Users might want to use blur intensity based on who they are communicating with.

* Background Blur API should work with [Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers).

## Non-goals

One adjacent goal was to combine Background Replacement with Background Blur as part of an overall Background Concealment API. Presently there's no platform APIs to support Background Replacement (green screen, animated gif, image, video). Combining both might be too premature. Instead, we would like to keep Background Replacement as a separate feature proposal for a later date.

## User research

Feedback on API shape.
* TEAMS
* ZOOM


## Background Blur API

This is about bringing platform background concealment capabilities to the web so constrainable media track capabilities fit naturally to the purpose. Because the concealment is (or should be) implemented by the platform media pipeline, it is enough for an (web) application to control the concealment through constrainable properties. The application does not have to (and actually cannot) do the actual concealment in this case. The concealment is already done before the application receives video frames. On Apple devices, Background Blur is controlled globally from Command Center and as such any app cannot independently switch on/off the feature. 

* backgroundBlur.idl
```js
partial dictionary MediaTrackSupportedConstraints {
  boolean backgroundBlur = true;
};

partial dictionary MediaTrackCapabilities {
  MediaSettingsRange backgroundBlur;
};

partial dictionary MediaTrackConstraintSet {
  MediaSettingsRange backgroundBlur;
};

partial dictionary MediaTrackSettings {
  double backgroundBlur;
};

```

## Example

 * main.js:
   ```js
   // main.js:
   // Open camera.
   const stream = navigator.mediaDevices.getUserMedia({video: true});
   const [videoTrack] = stream.getVideoTracks();

   // Use a video worker and show to user.
   const videoElement = document.querySelector('video');
   const videoWorker = new Worker('video-worker.js');
   videoWorker.postMessage({videoTrack: videoTrack}, [videoTrack]);
   const {data} = await new Promise(r => videoWorker.onmessage);
   videoElement.srcObject = new MediaStream([data.videoTrack]);
   ```
 * video-worker.js:
   ```js
   self.onmessage = async ({data: {videoTrack}}) => {
     const processor = new MediaStreamTrackProcessor({videoTrack});
     let readable = processor.readable;

     const capabilities = videoTrack.getCapabilities();
     if (capabilities.backgroundBlur && capabilities.backgroundBlur.max > 0) {
       // The platform supports background blurring.
       // Let's use platform background blurring.
       // No transformers are needed.
       await track.applyConstraints({
         advanced: [{backgroundBlur: capabilities.backgroundBlur.max}]
       });
     } else {
       // The platform does not support background blurring.
       // Let's use custom face detection to aid custom background blurring.
       importScripts('custom-face-detection.js', 'custom-background-blur.js');
       const transformer = new TransformStream({
         async transform(frame, controller) {
           // Use a custom face detection.
           const detectedFaces = await detectFaces(frame);
           // Use a custom background blurring.
           const newFrame = await blurBackground(frame, detectedFaces);
           frame.close();
           controller.enqueue(newFrame);
         }
       });
       // Pipe through a custom transformer.
       readable = readable.pipeThrough(transformer);
     };
     if (readable === processor.readable) {
       // No transformers were needed.
       // Pass the original track back to the main.
       parent.postMessage({videoTrack: videoTrack}, [videoTrack]);
     } else {
       // Transformers were needed.
       // Use a generator to generate a new video track and pass it to the main.
       const generator = new VideoTrackGenerator();
       parent.postMessage({videoTrack: generator.track}, [generator.track]);
       await readable.pipeTo(generator.writable);
     }
   };
   ```
[PR](https://github.com/w3c/mediacapture-extensions/pull/49)


## Security considerations

Background Blur feature would not expose any more security concerns compared to a video call without it. Since there's demand for Background Blur many products use a cloud based solution to satisfy conformance across a myriad of client devices. Modern clients are quite efficient these days to handle such popular tasks like Background Blur, either by leveraging AI accelerators or using specific vector instructions like AVX.

## Privacy considerations

Background Blur is supposed to enhance Privacy of the user compared to a video call without it. Many times users are in a video call where they do not know the audience well enough. It's advisable to not accidentally share any more personal information than required which might be otherwise be exposed without any form of Background Concealment. When in doubt, it's better to blur it out and users would change the blur intensity depending on who is on the other side of the call.

## Stakeholder Feedback / Opposition

[Implementors and other stakeholders may already have publicly stated positions on this work. If you can, list them here with links to evidence as appropriate.]

- [Teams] : No signals
- [Zoom] : No signals
- [WebEx] : No signals
- [Firefox] : No signals
- [Safari] : No signals

[If appropriate, explain the reasons given by other implementors for their concerns.]

## References & acknowledgements

Many thanks for valuable feedback and advice from:

- [Tuukka Toivonen]
- [Bernard Aboba]
- [Harald Alvestrand]
- [Jan-Ivar Bruaroey]
- [Youenn Fablet]
- [Dominique Hazael-Massieux]
- [etc.]
