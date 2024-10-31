---
id: 3wrqfar0uaz0v4pp4e62ta3
title: Packmule
desc: ''
updated: 1687977639976
created: 1687479679590
---
## [Packmule Boulangerie](https://github.com/helle253/packmule)

A rails app I put together for a buddy of mine. It is not currently live, unfortunately.

The most interesting part was implementing a client-side js rendering tool for taking a `.glb` file and outputting a nice GIF where the scene rotates endlessly. The intention was to be able to upload this and allow the content managers to upload a GIF every week for whatever the new boulangerie box was! You Cacan find the source code [here](https://github.com/helle253/packmule/blob/main/app/javascript/controllers/dashboard_controller.js). I wasn't satisfied with the existing solutions for consuming a scene description and returning a simple .gif (Cloudinary has )

CCapture.js allows us to preserve a constant frame-rate during the recording process, ensuring


![](assets/packmule.gif)

The cursor is missing in this gif, but hovering over one of the cards would pause the rotation!
