---
id: 3wrqfar0uaz0v4pp4e62ta3
title: Packmule
desc: ''
updated: 1687978572123
created: 1687479679590
---
## Packmule Boulangerie ([Source Code](https://github.com/helle253/packmule))

A rails app I put together for a buddy of mine. It is not currently live, unfortunately.

The most interesting part was implementing a client-side js rendering tool for taking a `.glb` file and outputting a nice GIF where the scene rotates endlessly. The intention was to be able to upload this and allow the content managers to upload a GIF every week for whatever the new boulangerie box was! You Cacan find the source code [here](https://github.com/helle253/packmule/blob/main/app/javascript/controllers/dashboard_controller.js). I wasn't satisfied with the existing solutions for consuming a scene description and returning a simple .gif. Cloudinary has [something](https://cloudinary.com/documentation/transformations_on_3d_models) like what I was looking for, but it did not support the minimum frame rates I was looking for.

CCapture.js allowed me to preserve a constant frame-rate during the recording process. User inputs the .glb, hits record, and the client-side browser spits something nice back out.

I wanted to add some primitive controls (view-angle, distance, rotation speed, duration, etc.) but working with Javascript from within Rails is pretty onerous, and the MVP got the job done.

![](assets/packmule.gif)

The cursor is missing in this snippet, but hovering over one of the cards would pause the rotation!

The rest of the app is fully functional, albeit sparing. Users can sign up and sign up for food-boxes. Admins can sign in, manage users, record notes about them (for recording delivery instructions), and upload media (with the intention of showcasing previous boxes).

It was an enjoyable experience building this, the folks I built it for were and are super collaborative partners. It required iteration and a ton of conversations honing in on what the goals were, and I'm proud of the finished product.
