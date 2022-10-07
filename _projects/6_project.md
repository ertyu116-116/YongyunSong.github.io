---
layout: page
title: Turtleneck detection
description: Senior_project
img: assets/img/senior_project/Turtle-neck.png
importance: 6
category: fun
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/senior_project/HR_net.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/senior_project/change_annotation.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    HR_net & modified annotation 
</div>

transfer learning - data 1,000
code application

```python
class Angle:
    def __init__(self,point1,point2,point3,point4):
        self.point1 = point1
        self.point2 = point2 
        self.point3 = point3 
        self.point4 = point4

    def xyztoangle(self):
        return math.degrees(math.atan2(self.point4-self.point2,self.point3-self.point1))    
    
    def distance(self):
        return math.sqrt((self.point4-self.point2)*(self.point4-self.point2)+(self.point3-self.point1)*(self.point3-self.point1))
```

* ## Result

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/senior_project/normal.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/senior_project/Turtle-neck.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    HR_net & modified annotation 
</div>