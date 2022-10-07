---
layout: page
title: Perspective Projection
description: with mono cameras
img: assets/img/perspective_projection/mono-camera_projection.png
importance: 1
category: work
---

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

#mono camera 5
fx =1286.3425
fy = 1282.2571
skew_c = 0
cx = 1287.1922
cy = 1035.7207
intrinsic_matrix = np.array([[fx,skew_c,cx],[0,fy,cy],[0,0,1]])
#extrinsic_matrix
extrinsic_matrix = np.array([[0.95917879, 0.14486955, -0.24287624, 2802.36],
                        [-0.14883925, 0.98885933, 0.0020263296, -371.48387],
                        [0.24046399, 0.034205906, 0.97005517, 605.537]]
                        )
camera_coord = np.matmul(extrinsic_matrix,img_homo.T) #(3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2,:,] #make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix,normalized_camera_coord) #(3,3) X (3,193910)

rgb = np.concatenate((r.T,g.T,b.T),axis=1)
rgb = rgb/256 # make 0-1 float
plt.subplot(3,4,3)
plt.title("mono camera 5")
scatter = plt.scatter(image_coord[0,:,],image_coord[1,:,],c=rgb)

#mono camera 7
fx =1288.0381
fy = 1213.7298
skew_c = 0
cx = 1288.4049
cy = 1047.8253
intrinsic_matrix = np.array([[fx,skew_c,cx],[0,fy,cy],[0,0,1]])

#extrinsic_matrix
extrinsic_matrix = np.array([[0.8583016, 0.2526078, -0.44666281, 4188.1098],
                        [-0.26920753, 0.9626996, 0.027143891 ,-612.10609],
                        [0.43685887, 0.09694735, 0.89429052,964.38422]]
                        )
camera_coord = np.matmul(extrinsic_matrix,img_homo.T) #(3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2,:,] #make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix,normalized_camera_coord) #(3,3) X (3,193910)

rgb = np.concatenate((r.T,g.T,b.T),axis=1)
rgb = rgb/256 # make 0-1 float
plt.subplot(3,4,4)
plt.title("mono camera 7")
scatter = plt.scatter(image_coord[0,:,],image_coord[1,:,],c=rgb)

#mono camera 9
fx =1284.3096
fy = 1242.4413
skew_c = 0
cx = 1284.4279
cy = 1042.1316
intrinsic_matrix = np.array([[fx,skew_c,cx],[0,fy,cy],[0,0,1]])

#extrinsic_matrix
extrinsic_matrix = np.array([[-0.99956982, 0.013956132, -0.025795499, 3364.4476],
                        [-0.016483159, 0.4601638, 0.88768101, -2651.0879],
                        [0.024258749, 0.88772434, -0.4597358, 4257.7071]]
                        )
camera_coord = np.matmul(extrinsic_matrix,img_homo.T) #(3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2,:,] #make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix,normalized_camera_coord) #(3,3) X (3,193910)

rgb = np.concatenate((r.T,g.T,b.T),axis=1)
rgb = rgb/256 # make 0-1 float
plt.subplot(3,4,5)
plt.title("mono camera 9")
scatter = plt.scatter(image_coord[0,:,],image_coord[1,:,],c=rgb)

#mono camera 11
fx =1292.0913
fy = 1215.9717
skew_c = 0
cx = 1291.9516
cy = 1030.2045
intrinsic_matrix = np.array([[fx,skew_c,cx],[0,fy,cy],[0,0,1]])


#extrinsic_matrix
extrinsic_matrix = np.array([[-0.96730441, -0.13599484, 0.21407379, 1896.9672],
                        [0.12675996, 0.47184531, 0.87252159, -2559.2354],
                        [-0.21966815, 0.87112996, -0.43917935, 4072.0264]]
                        )
camera_coord = np.matmul(extrinsic_matrix,img_homo.T) #(3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2,:,] #make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix,normalized_camera_coord) #(3,3) X (3,193910)

rgb = np.concatenate((r.T,g.T,b.T),axis=1)
rgb = rgb/256 # make 0-1 float
plt.subplot(3,4,6)
plt.title("mono camera 11")
scatter = plt.scatter(image_coord[0,:,],image_coord[1,:,],c=rgb)

# mono camera 13
fx = 1287.3334
fy = 1265.1114
skew_c = 0
cx = 1287.0891
cy = 1038.8095
intrinsic_matrix = np.array([[fx, skew_c, cx], [0, fy, cy], [0, 0, 1]])

# extrinsic_matrix
extrinsic_matrix = np.array([[-0.9504208, -0.15890217, 0.26730208, 529.28109],
                             [0.16130411, 0.48297702, 0.86064753, -2342.8452],
                             [-0.26585952, 0.86109424, -0.43339984, 3748.7594]]
                            )
camera_coord = np.matmul(extrinsic_matrix, img_homo.T)  # (3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2, :, ]  # make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix, normalized_camera_coord)  # (3,3) X (3,193910)

rgb = np.concatenate((r.T, g.T, b.T), axis=1)
rgb = rgb / 256  # make 0-1 float
plt.subplot(3, 4, 7)
plt.title("mono camera 13")
scatter = plt.scatter(image_coord[0, :, ], image_coord[1, :, ], c=rgb)

# mono camera 15
fx =1289.8211
fy = 1203.4967
skew_c = 0
cx = 1289.5996
cy = 1030.41
intrinsic_matrix = np.array([[fx, skew_c, cx], [0, fy, cy], [0, 0, 1]])

# extrinsic_matrix
extrinsic_matrix = np.array([[ -0.8475433,-0.26729894, 0.45849932, -769.46865],
                             [0.27472035, 0.51820665, 0.80993246, -2112.0755],
                             [-0.45409149, 0.81241193, -0.36577012, 3340.7309]]
                            )
camera_coord = np.matmul(extrinsic_matrix, img_homo.T)  # (3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2, :, ]  # make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix, normalized_camera_coord)  # (3,3) X (3,193910)

rgb = np.concatenate((r.T, g.T, b.T), axis=1)
rgb = rgb / 256  # make 0-1 float
plt.subplot(3, 4, 8)
plt.title("mono camera 15")
scatter = plt.scatter(image_coord[0, :, ], image_coord[1, :, ], c=rgb)

# mono camera 17
fx =1270.3472
fy = 1272.0425
skew_c = 0
cx = 1270.0608
cy = 1032.2583
intrinsic_matrix = np.array([[fx, skew_c, cx], [0, fy, cy], [0, 0, 1]])


# extrinsic_matrix
extrinsic_matrix = np.array([[0.78632702, -0.34202561, 0.51449811, -1307.5761],
                             [0.32192198, 0.93761757, 0.13129936, -203.16387],
                             [-0.52731021, 0.062384012 ,0.84737959 ,378.54639]]
                            )
camera_coord = np.matmul(extrinsic_matrix, img_homo.T)  # (3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2, :, ]  # make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix, normalized_camera_coord)  # (3,3) X (3,193910)

rgb = np.concatenate((r.T, g.T, b.T), axis=1)
rgb = rgb / 256  # make 0-1 float
plt.subplot(3, 4, 9)
plt.title("mono camera 17")
scatter = plt.scatter(image_coord[0, :, ], image_coord[1, :, ], c=rgb)

# mono camera 19
fx =1283.2529
fy = 1230.6382
skew_c = 0
cx = 1283.6855
cy = 1015.227
intrinsic_matrix = np.array([[fx, skew_c, cx], [0, fy, cy], [0, 0, 1]])


# extrinsic_matrix
extrinsic_matrix = np.array([[ 0.3495146, 0.52103313, -0.77869379 ,5116.9855],
                             [-0.50977551, 0.80307763, 0.30853728, -1096.4477],
                             [0.7861097, 0.28912074 ,0.54629729, 1826.9661]]
                            )
camera_coord = np.matmul(extrinsic_matrix, img_homo.T)  # (3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2, :, ]  # make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix, normalized_camera_coord)  # (3,3) X (3,193910)

rgb = np.concatenate((r.T, g.T, b.T), axis=1)
rgb = rgb / 256  # make 0-1 float
plt.subplot(3, 4, 10)
plt.title("mono camera 19")
scatter = plt.scatter(image_coord[0, :, ], image_coord[1, :, ], c=rgb)

# mono camera 21
fx =1296.0327
fy = 1251.0236
skew_c = 0
cx = 1295.6452
cy = 1047.8688
intrinsic_matrix = np.array([[fx, skew_c, cx], [0, fy, cy], [0, 0, 1]])


# extrinsic_matrix
extrinsic_matrix = np.array([[-0.84526811, 0.30032507, -0.44195777, 4274.3424],
                             [-0.28467871, 0.44684393, 0.8481088, -2526.5964],
                             [0.45219448, 0.84269529, -0.29220677, 4120.2694]]
                            )
camera_coord = np.matmul(extrinsic_matrix, img_homo.T)  # (3,4) X (4,193910) = (3,193910)
normalized_camera_coord = camera_coord / camera_coord[2, :, ]  # make homogeneous value 1
image_coord = np.matmul(intrinsic_matrix, normalized_camera_coord)  # (3,3) X (3,193910)

rgb = np.concatenate((r.T, g.T, b.T), axis=1)
rgb = rgb / 256  # make 0-1 float
plt.subplot(3, 4, 11)
plt.title("mono camera 21")
scatter = plt.scatter(image_coord[0, :, ], image_coord[1, :, ], c=rgb)

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
