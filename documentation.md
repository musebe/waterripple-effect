###  Nextjs waterripple effect

##  Introduction
This article is to demonstrate how to build a water ripple effect using Nextjs framework and  [Pixi.js](https://pixijs.com) library

##  Codesandbox 
The final version of this project can be viewed on   [Codesandbox](/).

<CodeSandbox
title="webrtc"
id=" "
/>

You can find the full source code on my [Github](/) repo.

##  Prerequisites

Basic/entry-level knowledge and understanding of javascript and React/Nextjs.

##  Setting Up the Sample Project

In your respective directory, Create a new Next.js project by using: `npx create-next-app videocall`

Go to your project directory  using: `cd videocall`

We will begin by setting up our backend with the [Cloudinary](https://cloudinary.com/?ap=em) feature.

Include [Cloudinary](https://cloudinary.com/?ap=em) in your dependencies: `npm install cloudinary`

###  Cloudinary Credentials Setup
Use [Link](https://cloudinary.com/console) to log in or create your Cloudinary account. You will be provided with the necessary environment variables for integration.

In your project root directory, create a new file named `.env.local` and use the following guide to fill your variables.
```
"pages/api/upload.js"


CLOUDINARY_CLOUD_NAME =

CLOUDINARY_API_KEY = 

CLOUDINARY_API_SECRET=

```

Restart your project: `npm run dev`.

Create a directory `pages/api/upload.js`.

Configure the environment keys and libraries to avoid code duplication.

```
"pages/api/upload.js"


var cloudinary = require("cloudinary").v2;

cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
});
```
Finally, add a handler function to execute Nextjs post request:
```
"pages/api/upload.js"


export default async function handler(req, res) {
    if (req.method === "POST") {
        let url = ""
        try {
            let fileStr = req.body.data;
            const uploadedResponse = await cloudinary.uploader.upload_large(
                fileStr,
                {
                    resource_type: "video",
                    chunk_size: 6000000,
                }
            );
        } catch (error) {
            res.status(500).json({ error: "Something wrong" });
        }

        res.status(200).json("backend complete");
    }
}
```
The above function will upload media files to Cloudinary and return the file's Cloudinary link as a response    

Let us now build the ripple effect.

First, include [Pixi.js](https://pixijs.com) which will allow us to create the effect using few lines of code: `npm install pixi.js`. The

Include all necessary imports in the Home component:

```
import React, { useRef, useState } from "react";
import * as PIXI from 'pixi.js';
```

Declare the following variables and state hooks. We will use them as we move on

```
"pages/index.js"


  let original, app, image, displacementSprite, displacementFilter, processedImage, animationID,finalCanvas;
  const processedRef = useRef();
  const [link, setLink] = useState();
```

Paste the following in your return statement. You can finc the css files in the github repository.

```
"pages/index.js"


return (
    <div className="container">
      <div className="navbar">
        <h1>WaterRipple Effect in nextjs</h1>
        <button onClick = {startAnimation}>Start water ripple</button>
        <button onClick = {stopAnimation}>Stop water ripple</button>
        {/* <button onClick={uploadHandler}>Upload Sample</button> */}
      </div>
      <div className="row">
        <div className="column">
          <img id="image" src="https://res.cloudinary.com/dogjmmett/image/upload/v1652412717/template_vowego.jpg" alt="fish" />
        </div>
        <div id="column2">
        {link?  <a href={link} className="link"><h3>Use Link</h3></a>:<h3 className="link">Link shows here</h3> }<br />

          <div ref={processedRef} className="processed" />
        </div>
      </div>
    </div>
)
```

The above should be the same as below:

![UI](https://res.cloudinary.com/dogjmmett/image/upload/v1652453122/UI_bnpct0.png "UI")

Create the following function named `startAnimation` and begin by refferencing the original image and the reffrenced div `processedRef` as follows:


```
"pages/index.js"


const startAnimation = () => {
    processedImage = processedRef.current;
    original = document.getElementById('image')
}
```
We can now initialize `Pixi.js` by first creating an application object that will render your scene like renderer, root container and time tracking. We then add the HTML as a canvas to the referenced `processedRef` DOM element.

```
"pages/index.js"


  const startAnimation = () => {
    processedImage = processedRef.current;
    original = document.getElementById('image');
    // console.log(original.width, original.height)
    app = new PIXI.Application({ width: original.width, height: original.height, forceCanvas: true });
    processedImage.appendChild(app.view);
 
  }

```
Intoduce a sprite image.

```
"pages/index.js"


  const startAnimation = () => {
    processedImage = processedRef.current;
    original = document.getElementById('image');
    // console.log(original.width, original.height)
    app = new PIXI.Application({ width: original.width, height: original.height, forceCanvas: true });
    processedImage.appendChild(app.view);
    image = new PIXI.Sprite.from("https://res.cloudinary.com/dogjmmett/image/upload/v1652412717/template_vowego.jpg");
    image.width = original.width;
    image.height = original.height;
    // console.log(app.view)
    app.stage.addChild(image);
 
  }
```

The trick to creating the displacement effect is to move the image pixels away from their original position. To achieve such displacement I will map the original image with a random black and white variation 

![blackWhite](https://res.cloudinary.com/dogjmmett/image/upload/v1652453120/cloud_dt6mr6.png "blackWhite")

Create sprite from this image and add a displacement filter from the sprite. Set wrapMode to repeat and displacement to cover the entire image 

 ```
 "pages/index.js"


  const startAnimation = () => {
    processedImage = processedRef.current;
    original = document.getElementById('image');
    // console.log(original.width, original.height)
    app = new PIXI.Application({ width: original.width, height: original.height, forceCanvas: true });
    processedImage.appendChild(app.view);
    image = new PIXI.Sprite.from("https://res.cloudinary.com/dogjmmett/image/upload/v1652412717/template_vowego.jpg");
    image.width = original.width;
    image.height = original.height;
    // console.log(app.view)
    app.stage.addChild(image);

    displacementSprite = new PIXI.Sprite.from("https://res.cloudinary.com/dogjmmett/image/upload/v1652411299/cloud_eg89xa.png")
    displacementFilter = new PIXI.filters.DisplacementFilter(displacementSprite);
    displacementSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
    app.stage.addChild(displacementSprite);
    app.stage.filters = [displacementFilter];
 
}
```

Add an animation for a better view of the displacement effect and set it on a loop using `requestAnimationFrame` method.

```
"pages/index.js"


function animate() {
    displacementSprite.x += 10;
    displacementSprite.y += 4;
    animationID = requestAnimationFrame(animate);
    finalCanvas = app.view.toDataURL()

}
```
A sample of captioned effect looks like below:

![Rippled]( https://res.cloudinary.com/dogjmmett/image/upload/v1652452132/tp1smstun6fhoex3ux76.png "Rippled")

That's it! Ensure to go through this article to enjoy your experience.