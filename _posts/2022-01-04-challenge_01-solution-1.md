---
date:   2022-01-04 11:13:57 +0200 #mandatory for Jekyll
layout: layout_solution
permalink: "challenge_01-solution.html"
assets_folder: "challenge_01/solution_01"
in_list__title:  "Challenge 1 [Solution]"
post_type: "solution"
credit_line: "A solution by [RGgt](https://github.com/RGgt)"
challenge: "challenge-01.html"
og_title: "Flutter Challenges: Challenge 01 [Solution using ClipPath and CustomClipper]"
og_description: "To implement this design, we will need to place 2 widgets, one on top of the other. One widget will be a simple box with a white background; the other widget will have a purple background and will later be clipped to give the wave-like appearance..."
og_image: "assets/{{ page.assets_folder }}/og_challenge_01_solution.jpg"

categories: jekyll update # ?
---
{% include solution_header_template.html  %}


To implement this design, we will need to place 2 widgets, one on top of the other. One widget will be a simple box with a white background; the other widget will have a purple background and will later be clipped to give the wave-like appearance.

For simplicity, I will initially use some hard-coded values for sizes.

```dart
import 'package:flutter/widgets.dart';

class WaveWidget extends StatelessWidget {
  const WaveWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // The white box
        SizedBox(
            width: 300,
            height: 500,
            child: Container(
                color: const Color.fromARGB(255, 230, 230, 255),
                padding: const EdgeInsets.all(10),
                child: const Text("Lorem Ipsum Text",
                    style: TextStyle(fontWeight: FontWeight.bold)))),
        // The purple box
        Positioned(
          bottom: 0,
          child: SizedBox(
            width: 300,
            height: 450,
            child: Container(
                color: const Color.fromARGB(255, 98, 0, 158),
                padding: const EdgeInsets.fromLTRB(
                    10, 10, 10, 10),
                child: const Text(
                "Lorem ipsum dolor sit amet, consectetur ...",
                style: TextStyle(color: Color.fromARGB(255, 230, 230, 255)),
                )
            ),
          ),
        ),
      ],
    );
  }
}
```
 

The result will look like this:
<div align="center">

<img src="assets/{{ page.assets_folder }}/sol_simple_boxes.jpg" height="600">

</div>

Now we need to click the purple box and give it the wave-like aspect.

To do that, we will wrap its **Container** element (which is actually painted purple) in a <strong><a href="https://api.flutter.dev/flutter/widgets/ClipPath-class.html" target="_blank">ClipPath</a></strong> widget, to which we will provide a custom **clipper**.

```dart
        ...
        // The purple box
        Positioned(
          bottom: 0,
          child: SizedBox(
            width: 300,
            height: 450,
            // Wrap the container inside a ClipPath
            child: ClipPath(
              // And provide a custom clipper
              clipper: MyClipper(),
              child: Container(
                  color: const Color.fromARGB(255, 98, 0, 158),
                  padding: const EdgeInsets.fromLTRB(
                      10, 10, 10, 10),
                  child: const Text(
                  "Lorem ipsum dolor sit amet, consectetur ...",
                  style: TextStyle(color: Color.fromARGB(255, 230, 230, 255)),
                  )
              ),
            ),
          ),
        ),
        ...
```
 
Our custom clipper will extend <strong><a href="https://api.flutter.dev/flutter/rendering/CustomClipper-class.html" target="_blank">CustomClipper&lt;Path&gt;</a></strong> and override <strong><a href="https://api.flutter.dev/flutter/rendering/CustomClipper/getClip.html" target="_blank">getClip</a></strong> and <strong><a href="https://api.flutter.dev/flutter/rendering/CustomClipper/shouldReclip.html" target="_blank">shouldReclip</a></strong>. 

In <strong><a href="https://api.flutter.dev/flutter/rendering/CustomClipper/shouldReclip.html" target="_blank">shouldReclip</a></strong> we only need to tell Flutter if we want to apply the clip again. As in our first implementation of the clipper, we will have no parameters, so it is ok to always return false for now.

The interesting part is <strong><a href="https://api.flutter.dev/flutter/rendering/CustomClipper/getClip.html" target="_blank">getClip</a></strong>, where we will define a <strong><a href="https://api.flutter.dev/flutter/dart-ui/Path-class.html" target="_blank">Path</a></strong> that will give the container the wave-like shape. 

In fact, the <strong><a href="https://api.flutter.dev/flutter/dart-ui/Path-class.html" target="_blank">Path</a></strong> that you return from this function will actually represent the area of the clipped control that you want to be visible. Anything that will be outside the path will be clipped out. 

By default, a path starts at the (0,0) point, but we need to start at a bit lower, at the base of the first wave. So, if the wave height is 25, we first need to move the starting point to (0, -25).

From there, we need to draw a curve line for the first wave. This wave should end at (width/2, -25) and from there will start the second wave, which will end at (width, -25).

These two waves will be defined using the <strong><a href="https://api.flutter.dev/flutter/dart-ui/Path/quadraticBezierTo.html" target="_blank">quadraticBezierTo</a></strong> function. The first two parameters of this function define the control point, while the last two define the ending point of the curve. You will need to experiment with the control point for a bit, as it will not be "a point on the curve." Rather, it will act as a sort of magnet.

After you have defined the two waves, the only thing left is to include the rectangular part below them in the path.

For this, we use the simple function <strong><a href="https://api.flutter.dev/flutter/dart-ui/Path/lineTo.html" target="_blank">lineTo</a></strong>.

```dart
class MyClipper extends CustomClipper<Path> {
  MyClipper();

  @override
  getClip(Size size) {
    var path = Path();
    // Move the the base of the first wave
    path.moveTo(0, 25);
    // The first wave (oriented upward)
    path.quadraticBezierTo(size.width / 5, -25, size.width / 2, 25);
    // The second wave (oriented downward)
    path.quadraticBezierTo(5 * size.width / 6, 3 * 25, size.width, 25);
    // Bottom-Right of the Container
    path.lineTo(size.width, size.height);
    // Bottom-Left of the container
    path.lineTo(0, size.height);
    path.close();
    return path;
  }

  @override
  bool shouldReclip(MyClipper oldClipper) {
    return true;
  }
}
```

The screen is starting to look as it should.

<div align="center">

<img src="assets/{{ page.assets_folder }}/sol_waves_01.png" height="600">

</div>

The first obvious issue is that the text is also partially clipped out. To fix this, we need to change the padding of the purple rectangle so that its content is placed below the lowest point of the second wave.

So keeping the wave height hard-coded as 25, the code becomes:
```dart
        ...
        // The purple box
        Positioned(
          bottom: 0,
          child: SizedBox(
            width: 300,
            height: 450,
            child: ClipPath(
              clipper: MyClipper(),
              child: Container(
                  color: const Color.fromARGB(255, 98, 0, 158),
                  padding: const EdgeInsets.fromLTRB(
                      10, 10 + 2 * 25 /* account for waves height */, 10, 10),
                  child: const Text(
                  "Lorem ipsum dolor sit amet, consectetur ...",
                  style: TextStyle(color: Color.fromARGB(255, 230, 230, 255)),
                  )
              ),
            ),
          ),
        ),
        ...
```
At this point, we have a functional screen that looks like it should.

<div align="center">

<img src="assets/{{ page.assets_folder }}/sol_waves_02.png" height="600">

</div>

The next step will be to replace all hard-coded values with parameters, but this is mainly simple Flutter programming. I will not do this here as it will needlessly complicate the code. So before concluding, let's take a final look at the complete code that deals with the main aspect of the challenge:

```dart
import 'package:flutter/widgets.dart';

class WaveWidget extends StatelessWidget {
  const WaveWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        // The white box
        SizedBox(
          width: 300,
          height: 500,
          child: Container(
            color: const Color.fromARGB(255, 230, 230, 255),
            padding: const EdgeInsets.all(10),
            child: const Text(
              "Lorem Ipsum Text",
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
        ),
        // The purple box
        Positioned(
          bottom: 0,
          child: SizedBox(
            width: 300,
            height: 450,
            child: ClipPath(
              clipper: MyClipper(),
              child: Container(
                color: const Color.fromARGB(255, 98, 0, 158),
                padding: const EdgeInsets.fromLTRB(10, 10 + 2 * 25, 10, 10),
                child: const Text(
                  "Lorem ipsum dolor sit amet, consectetur ...",
                  style: TextStyle(color: Color.fromARGB(255, 230, 230, 255)),
                ),
              ),
            ),
          ),
        ),
      ],
    );
  }
}

class MyClipper extends CustomClipper<Path> {
  MyClipper();

  @override
  getClip(Size size) {
    var path = Path();
    path.moveTo(0, 25);
    path.quadraticBezierTo(size.width / 5, -25, size.width / 2, 25);
    path.quadraticBezierTo(5 * size.width / 6, 3 * 25, size.width, 25);
    path.lineTo(size.width, size.height);
    path.lineTo(0, size.height);
    path.close();
    return path;
  }

  @override
  bool shouldReclip(MyClipper oldClipper) {
    return true;
  }
}

```

## Summary
<span class="bold">1. We created a Stack with the white widget at the bottom and the purple one on top. </span><br/>
<span class="bold">2. We added a ClipPath wrapper to the purple widget and supplied it with a custom clipper. </span><br/>
<span class="bold">3. In our custom clipper, we returned the area that we wanted to remain visible, and everything else was cut out. We experimented with the control point of the Bezier curves until we were pleased with the visual result.</span><br/>

Now all that is left is to replace the hard-coded values with parameters.
 
<hr/>
 
<a href="https://dartpad.dev/?id=63c89386e7b2c9b5e0b37b188c183070" target="_blank">See the basic solution</a> on DartPad


<a href="https://dartpad.dev/?id=03e0bc5759e05e6652b6855f4031fe97" target="_blank">See a slightly more polished impementation of the  solution</a> on DartPad
