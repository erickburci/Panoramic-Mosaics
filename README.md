![image.png](attachment:image.png)

# Creating Image Mosaics using Homographies 

In this project, I will align and blend together multiple photographs 
to form a panorama mosaic that extends the field of view of a camera.

When two images are taken with same center-of-projection, they can be aligned with a
homography transformation. To estimate the correct warping, I will need at
least 4 pairs of corresponding points in the overlaping region. I will mark these
points manually and compute a transformation that aligns them.

