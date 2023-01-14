---
layout: distill
title: Streo Vision
description: Characteristics of Stereo Vision
date: 2022-01-14

# authors:
#   - name: Albert Einstein
#     url: "https://en.wikipedia.org/wiki/Albert_Einstein"
#     affiliations:
#       name: IAS, Princeton
#   - name: Boris Podolsky
#     url: "https://en.wikipedia.org/wiki/Boris_Podolsky"
#     affiliations:
#       name: IAS, Princeton
#   - name: Nathan Rosen
#     url: "https://en.wikipedia.org/wiki/Nathan_Rosen"
#     affiliations:
#       name: IAS, Princeton

# bibliography: 2018-12-22-distill.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Methods of 3D reconstruction
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Disparity
  - name: Multiple views
  - name: Stereo Vision
  - name: Reference

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }

---

- 3D reconstruction
    - Shading, focus/defocus, texture, perspective effects, motion, occlusion, stereo
- Human stereopsis
    - fixation, disparity
- Multiple views
    - epipolar geometry
- Stereo Vision
    - geometry of a parallel stereo system
    - correspondence

## Methods of 3D reconstruction

* ### Dimensionality Reduction 

  The spatial information is lost while 3D object is projected to 2D. In that case, lengths are lost and angle preservation is lost. Hence, Parallel line are lost but there is a vanishing point where two lines, which was once parallel line, meet together.

  From this vanishing point, we can find out and track the spatial information lost.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Dimensinality_Reduction.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure1. Dimensinality reduction
</div>


* ### Structure from Shadow

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Shape_from_Shading.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure2. Structure from Shadow
</div>



* Lmertian Reflectance Model for approximating the reflectance property of the diffuse surface

$$
Diffuse\; Surface\; Color = \frac{\rho_d}{\pi} *L_i * cos{\theta}
$$

Light from all points on the surface reaches the viewer (camera). It means we can deduce the 3D surface from intensity and angles.

* ### Structure from focus

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Shape_from_Focus.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure3. Structure from focus
</div>

Despite the same images, we can deduce 3D information while using low-pass filter with different sigma to make images blur or not.

* ### Texture

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Shape_from_Texture.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure4. Structure form Texture
</div>

We can infer the 3D data while measuring images pattern. e.g. If the white seeds of strawberry is close, it means the distance is close. If the white seeds of strawberry is far, it means the distance is far. 

* ### Perspective effects

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Vanishing_point.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure5. Vanishing point
</div>

Perspective effect means even paraller line converge on one point while 3D image is prjected to 2D image. We can infer spatial information from vanishing point. the side effect of this method,However, is there is foriegn objects of air and the signal is hard to reach on camera. e.g.) when we take a mountain photo, the more the object is far from the camera, the more ambiguous the object is. 

* ### Stereo 

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Stereo_Vision.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure6. Stereo Vision
</div>

It is important to note that the camera centor should be moved not fixed with rotation in Stereo Vision for obtaining exact 3D information. 


## Disparity

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Disparity.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure7. Human steropsis: disparity
</div>

disparity occurs when eyes fixate on one object; others appear at different visual angles. Disparity is distance from b1 to b2 along retina.

## Multiple views

Why we have multiple views? Because there is distortion when projection occurs. e.g. the line which is connected the optical center of camera is projected to one point, not line.  

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Multiple_views.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure8. Multiple views
</div>

In multiple views, we shift the projection image to infront of the optical center than using pin hole model which change the locations for 2D coordinate,as an ideal mathematic method.

## Stereo Vision

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Stereo Vision/Epipolar_Geometry.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure9. Epipolar Geometry
</div>

epipole - when the line which connects optical centers of two cameras, e and e' are called to epipoles.
epipolar line - the line which connects epipole and point projected. 
epipolar constraint - Depending on DOF(Degree of Freedom), there are the 8,7,5,3 and 1 pair of matching points to obtain fundamental and essential matrix. 
Triangulation - If we know E, F matrix and p,p', we can obtain 3D point.


## Reference

* https://www.youtube.com/watch?v=3QjEOlfvg9M&t=2326
* https://math.hws.edu/graphicsbook/c4/s1.html