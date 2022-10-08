---
layout: page
title: Perspective Projection
description: with mono cameras
img: assets/img/perspective_projection/mono-camera_projection.png
importance: 1
category: research
---
* ## 3D point cloud image progection to 2D image (for removing background withsegmentation)

```python
#!/usr/bin/env python3.4

from open3d import *
import os
import re
import numpy as np
import uuid
from scipy import misc
import numpy as np
from PIL import Image
import sys
import imageio
import matplotlib.pyplot as plt
import cv2
from plyfile import PlyData, PlyElement

def read(file):
    if file.endswith('.float3'): return readFloat(file)
    elif file.endswith('.flo'): return readFlow(file)
    elif file.endswith('.ppm'): return readImage(file)
    elif file.endswith('.pgm'): return readImage(file)
    elif file.endswith('.png'): return readImage(file)
    elif file.endswith('.jpg'): return readImage(file)
    elif file.endswith('.pfm'): return readPFM(file)[0]
    else: raise Exception('don\'t know how to read %s' % file)

def write(file, data):
    if file.endswith('.float3'): return writeFloat(file, data)
    elif file.endswith('.flo'): return writeFlow(file, data)
    elif file.endswith('.ppm'): return writeImage(file, data)
    elif file.endswith('.pgm'): return writeImage(file, data)
    elif file.endswith('.png'): return writeImage(file, data)
    elif file.endswith('.jpg'): return writeImage(file, data)
    elif file.endswith('.pfm'): return writePFM(file, data)
    else: raise Exception('don\'t know how to write %s' % file)

def readPFM(file):
    file = open(file, 'rb')

    color = None
    width = None
    height = None
    scale = None
    endian = None

    header = file.readline().rstrip()
    if header.decode("ascii") == 'PF':
        color = True
    elif header.decode("ascii") == 'Pf':
        color = False
    else:
        raise Exception('Not a PFM file.')

    dim_match = re.match(r'^(\d+)\s(\d+)\s$', file.readline().decode("ascii"))
    if dim_match:
        width, height = list(map(int, dim_match.groups()))
    else:
        raise Exception('Malformed PFM header.')

    scale = float(file.readline().decode("ascii").rstrip())
    if scale < 0: # little-endian
        endian = '<'
        scale = -scale
    else:
        endian = '>' # big-endian

    data = np.fromfile(file, endian + 'f')
    shape = (height, width, 3) if color else (height, width)

    data = np.reshape(data, shape)
    data = np.flipud(data)
    return data, scale

def writePFM(file, image, scale=1):
    file = open(file, 'wb')

    color = None

    if image.dtype.name != 'float32':
        raise Exception('Image dtype must be float32.')

    image = np.flipud(image)

    if len(image.shape) == 3 and image.shape[2] == 3: # color image
        color = True
    elif len(image.shape) == 2 or len(image.shape) == 3 and image.shape[2] == 1: # greyscale
        color = False
    else:
        raise Exception('Image must have H x W x 3, H x W x 1 or H x W dimensions.')

    file.write('PF\n' if color else 'Pf\n'.encode())
    file.write('%d %d\n'.encode() % (image.shape[1], image.shape[0]))

    endian = image.dtype.byteorder

    if endian == '<' or endian == '=' and sys.byteorder == 'little':
        scale = -scale

    file.write('%f\n'.encode() % scale)

    image.tofile(file)

def readFlow(name):
    if name.endswith('.pfm') or name.endswith('.PFM'):
        return readPFM(name)[0][:,:,0:2]

    f = open(name, 'rb')

    header = f.read(4)
    if header.decode("utf-8") != 'PIEH':
        raise Exception('Flow file header does not contain PIEH')

    width = np.fromfile(f, np.int32, 1).squeeze()
    height = np.fromfile(f, np.int32, 1).squeeze()

    flow = np.fromfile(f, np.float32, width * height * 2).reshape((height, width, 2))

    return flow.astype(np.float32)

def readImage(name):
    if name.endswith('.pfm') or name.endswith('.PFM'):
        data = readPFM(name)[0]
        if len(data.shape)==3:
            return data[:,:,0:3]
        else:
            return data

    return imageio.imread(name)

def writeImage(name, data):
    if name.endswith('.pfm') or name.endswith('.PFM'):
        return writePFM(name, data, 1)

    return imageio.imsave(name, data)

def writeFlow(name, flow):
    f = open(name, 'wb')
    f.write('PIEH'.encode('utf-8'))
    np.array([flow.shape[1], flow.shape[0]], dtype=np.int32).tofile(f)
    flow = flow.astype(np.float32)
    flow.tofile(f)

def readFloat(name):
    f = open(name, 'rb')

    if(f.readline().decode("utf-8"))  != 'float\n':
        raise Exception('float file %s did not contain <float> keyword' % name)

    dim = int(f.readline())

    dims = []
    count = 1
    for i in range(0, dim):
        d = int(f.readline())
        dims.append(d)
        count *= d

    dims = list(reversed(dims))

    data = np.fromfile(f, np.float32, count).reshape(dims)
    if dim > 2:
        data = np.transpose(data, (2, 1, 0))
        data = np.transpose(data, (1, 0, 2))

    return data

def writeFloat(name, data):
    f = open(name, 'wb')

    dim=len(data.shape)
    if dim>3:
        raise Exception('bad float file dimension: %d' % dim)

    f.write(('float\n').encode('ascii'))
    f.write(('%d\n' % dim).encode('ascii'))

    if dim == 1:
        f.write(('%d\n' % data.shape[0]).encode('ascii'))
    else:
        f.write(('%d\n' % data.shape[1]).encode('ascii'))
        f.write(('%d\n' % data.shape[0]).encode('ascii'))
        for i in range(2, dim):
            f.write(('%d\n' % data.shape[i]).encode('ascii'))

    data = data.astype(np.float32)
    if dim==2:
        data.tofile(f)

    else:
        np.transpose(data, (2, 0, 1)).tofile(f)

os.chdir("/home/pub/db/Dataset_20190805_set14_PM/Depth")#change path

a= read('Pod1_Depth_000.png') #depth

os.chdir("/home/pub/db/Dataset_20190805_set14_PM/Color")

b= read('Pod1_Color_000.png') #color

os.chdir("/home/pub/db/Dataset_20190805_set14_PM/PC_R")

plydata = PlyData.read('pc_000.ply')

c = plydata.elements[0].data['x']
d = plydata.elements[0].data['y']
e = plydata.elements[0].data['z']

x = np.array([c])
y = np.array([d])
z = np.array([e])


r = np.array([plydata.elements[0].data['red']])
g = np.array([plydata.elements[0].data['green']])
b = np.array([plydata.elements[0].data['blue']])

img_xyz = np.concatenate((x.T,y.T,z.T),axis=1) #(193910,3)

new_axis = np.ones(shape=(img_xyz.shape[0],1))
img_homo = np.concatenate((img_xyz,new_axis),axis=1)


fig, ax = plt.subplots(3, 4)

#mono camera 1
#intrinsic_matrix
fx = 1291.7711
fy = 1259.1579
skew_c = 0
cx = 1291.9264
cy = 1038.7288
intrinsic_matrix = np.array([[fx,skew_c,cx],[0,fy,cy],[0,0,1]])
#extrinsic_matrix
extrinsic_matrix = np.array([[1,0,0,-0],
                        [0,1,0,-0],
                        [0,0,1,-0]]
                        )
camera_coord = np.matmul(extrinsic_matrix,img_homo.T) #(3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2,:,] #make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix,normalized_camera_coord) #(3,3) X (3,193910)

rgb = np.concatenate((r.T,g.T,b.T),axis=1)
rgb = rgb/256 # make 0-1 float
plt.subplot(3,4,1)
plt.title("mono camera 1")
scatter = plt.scatter(image_coord[0,:,],image_coord[1,:,],c=rgb)

#mono camera 3
fx =1309.6586
fy = 1243.5969
skew_c = 0
cx = 1309.6512
cy = 1023.6217
intrinsic_matrix = np.array([[fx,skew_c,cx],[0,fy,cy],[0,0,1]])
#extrinsic_matrix
extrinsic_matrix = np.array([[0.96179149,0.14086911,-0.23476163,1350.3909],
                        [-0.14401313,0.98956853,0.0037869379,-172.98239],
                        [0.23284619,0.030166514,0.97204559,252.05389]]
                        )
camera_coord = np.matmul(extrinsic_matrix,img_homo.T) #(3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2,:,] #make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix,normalized_camera_coord) #(3,3) X (3,193910)

rgb = np.concatenate((r.T,g.T,b.T),axis=1)
rgb = rgb/256 # make 0-1 float
plt.subplot(3,4,2)
plt.title("mono camera 3")
scatter = plt.scatter(image_coord[0,:,],image_coord[1,:,],c=rgb)

...

# mono camera 23
fx =1294.3257
fy = 1269.2912
skew_c = 0
cx =  1293.462
cy = 1036.3195
intrinsic_matrix = np.array([[fx, skew_c, cx], [0, fy, cy], [0, 0, 1]])


# extrinsic_matrix
extrinsic_matrix = np.array([[-0.31314612, -0.51822336, 0.79585429, -1966.3649],
                             [0.50355057, 0.61990892 ,0.60178879, -1766.1489],
                             [-0.80521819 ,0.58920071, 0.066829594, 2843.7565]]
                            )
camera_coord = np.matmul(extrinsic_matrix, img_homo.T)  # (3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2, :, ]  # make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix, normalized_camera_coord)  # (3,3) X (3,193910)

rgb = np.concatenate((r.T, g.T, b.T), axis=1)
rgb = rgb / 256  # make 0-1 float
plt.subplot(3, 4, 12)
plt.title("mono camera 23")
scatter = plt.scatter(image_coord[0, :, ], image_coord[1, :, ], c=rgb)


plt.show()
```

* ## concatenate two image, depth image and rgb iamge

<div class="row">
    <div class="col-sm - 3 mt mt-md-0">
        {% include figure.html path="assets/img/perspective_projection/depth.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm - 3 mt mt-md-0">
        {% include figure.html path="assets/img/perspective_projection/rgb_image.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>    
</div>
<div class="caption">
    Figure 1. Left is depth image, right is rgb image augmented by eliminating backrounds.
</div>      

