
![Screenshot 2024-03-13 172546](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/dff54625-3643-4937-a724-88b48a9ead9f)

# Creating Image Mosaics using Homographies 

***see PanoramicMosaics.pynb file for code***

In this project, I will align and blend together multiple photographs 
to form a panorama mosaic that extends the field of view of a camera.

When two images are taken with same center-of-projection, they can be aligned with a
homography transformation. To estimate the correct warping, I will need at
least 4 pairs of corresponding points in the overlaping region. I will mark these
points manually and compute a transformation that aligns them.

# Part 1. Point Correspondence

The first image in each example is the central image.  It's simplest
to construct a mosaic from a central image and a set of peripheral images,
since we need to find just one homography for each peripheral image. The code 
in Part 1 is to allow the user to manually select at least
4 pairs of correpsonding points between each central and peripheral image. 
These points should be located on distinctive locations that the can easily be
identified between the images such as high contrast corners.

The code in Part 1 loads in the central image and then loops over the
remaining images and for each image which allows users to select four or more points.
The code then saves out the resulting points to a pckl file. 

The file **selectpoints.py** contains code that takes as input the plot axis and the number of points 
I want from the user. As the user clicks, the points are numbered so that we can 
make sure points correspond in the two images we are trying to align.

The following demonstrate the functionality of the get_correspondences function. As noted above, they should be executed one at a time. First execute the first cell which should display two pairs of images and get the user clicks. Once you have finished clicking, execute the second cell which grabs the point coordinates where the user clicked and saves the results out to disk. The third cell demonstrates loading the data back in from disk and visualizing the points again.

![Screenshot 2024-03-13 171554](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/ac6725cb-e69d-431b-ab3f-40c48c4c6da8)
![Screenshot 2024-03-13 171533](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/c2cf0e09-6470-4a49-b30c-53c7d83531c2)
![Screenshot 2024-03-13 171510](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/399574e3-dbc4-4506-9714-da42f1fa1fa5)
![Screenshot 2024-03-13 171456](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/315a88fe-f138-4317-8ca4-60732f68ed04)
![Screenshot 2024-03-13 171442](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/1280a58d-0ee9-4724-9722-833174c6d833)

# Part 2. Homography Transformations

For each image, we will need to compute the homography (3x3 transformation matrix) 
using *linear least squares*. This transformation should map the points in the 
peripheral image that you clicked to their corresponding points in the base 
"central" image. For the central image itself, this transformation would just be 
the identity matrix.

The first function, **compute_homography** should estimate a transformation matrix H given the pairs of points.  
The second function, **apply_homography** should take as input an array of point coordinates and a 3x3 matrix 
and return the transformed coordinates.

Note that if the matrix <tt>H</tt> maps (x1,y1) to (x2,y2), then the inverse
mapping is given simply by inverting the matrix.  So applying the homography
<tt>inv(H)</tt> will map (x2,y2) back to (x1,y1).

# Part 3. Warping

The function called **warp_images** takes the collection of correspondences
and generates warped versions of all the input images to align them with the final mosaic.

1. We will use the central image's coordinate system for the final mosaic.  We
first need to figure out how big the final mosiac will be.  We can accomplish
this by determining where the corners of each source image will be mapped to in
the final mosaic (using **apply_homography** function) and then use min/max to
determine the left-most, right-most, top-most and bottom-most points across all
of the warped images.    After this step we will have determined that all the
warped image pixels from all the images will fall inside some rectangular region
*(xmin,ymin)-(xmax,ymax)*.  Note that these coordinates will be expressed with respect 
to our central image. For example, *xmin* will be a negative value if some of the images
in our mosaic are mapped to the left of our central image.<p>

2. Generate the coordinates of all the pixels for our final mosaic as well as the
coordinates of pixels in each source image. 
To get the warped image coordinates, apply the estimated homography to the source
image pixel coordinates to determine where they will fall in the output mosaic.<p>

3. To produce the warped image, we will use **scipy.interpolate.griddata** to 
perform interpolation gray values onto a regular grid. We need to provide **griddata** 
with three pieces of information: the coordinates of the pixels after we have warped 
them with the appropriate homography, the gray value for each of those pixels, and 
the grid of pixel coordinates for our final mosaic. We will ultimately call this warping 
function for each source image, resulting in a new warped image the size of the final 
mosaic containing. By default, **griddata** will set the value of any pixels that are 
outside the source image to NaN. The figure below shows examples of warped images where 
the white pixels correspond to regions outside the source image (i.e. filled with NaNs).<p>

# Part 4. Blending

Now that we have generated the individual warped images, we need to blend
them together into the final mosaic image.  The simplest approach is to paste 
down the pixels from each warped image in some order into the output image. 
However, this can lead to bad artifacts.  Instead we 
should create a smooth blend between the images in the regions where they 
overlap. 

To create a blend, first compute an alpha mask for each image which is the same
size as the target output with 1s where you have values from that image. I start by useing **np.isnan** to get a boolean mask for each warped image that 
tells us which pixels are valid and which are invalid (were outside the source 
image).  In order to turn the binary mask into an alpha mask, we can feather 
the edges by bluring them with a Gaussian filter. Since more than one image can overlap at a 
given location, the final step is to normalize these 
alpha maps by the sum of the alphas across all images at that location.

To create the Gaussian filter,**gaussian_filter** takes an argument sigma which
specifies the width of the Gaussian. I needed to experiment with the 
parameter sigma in order to get good featering of the edges. 

# Results
-------------------------------------------------------------------------------------------------------
## Atrium Panorama
![Screenshot 2024-03-13 171926](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/a37c76db-ab15-4a87-b315-637183ac0526)
![Screenshot 2024-03-13 171916](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/870aa918-372e-4a5a-b721-dbd33b0bc446)
![Screenshot 2024-03-13 171905](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/03558a76-55ab-4199-b453-c7ca76977733)

## Library
![Screenshot 2024-03-13 172212](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/c0e173f0-f890-43e4-afa0-8171e9a857ed)
![Screenshot 2024-03-13 172202](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/73d8fc80-e7ad-41b7-9608-adf1b3861fd4)
![Screenshot 2024-03-13 172155](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/5999e341-8afe-461d-8df8-81ec022eb778)
![Screenshot 2024-03-13 172146](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/120cd5ca-227f-4881-b92e-3844fd0d5317)
![Screenshot 2024-03-13 172139](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/b0c520b3-c37e-4a0a-8417-d13870a507d3)
![Screenshot 2024-03-13 172125](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/98ad4c25-50ac-409d-8df0-0b52c87e8ca1)

## Apartment
![Screenshot 2024-03-13 172402](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/dba0dc74-9a10-413d-8ca3-bc7d1046c591)
![Screenshot 2024-03-13 172354](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/e7e8482c-1b5f-4b4f-b181-d9b537b5562c)
![Screenshot 2024-03-13 172347](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/7d6c261b-5436-4583-92b7-68801bab3718)
![Screenshot 2024-03-13 172341](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/211ee5c2-253a-4914-bccf-41b287312161)
![Screenshot 2024-03-13 172407](https://github.com/erickburci/Panoramic-Mosaics/assets/159087967/fcf6800e-5a9c-417f-9943-7cab991b4749)









