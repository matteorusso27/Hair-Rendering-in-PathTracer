# Hair Rendering in PathTracer
Developed by Matteo Russo 1664715 and Paolo Tarantino 1666228 realized for the Fundamentals of Computer Graphics 
course(AY 2019/2020) in Università degli Studi di Roma La Sapienza.

# Introduction
The project has the goal to provide to the yocto library the hair scattering implementation
presented in Matt Pharr's paper, one of the author of the pbrt book. The yocto libray (https://github.com/xelatihy/yocto-gl) is owned 
by our Professor Fabio Pellacini.

The yocto renderer treats hair as a surface. While this could be a fair approximation
in some cases, the paper's approach treats it as a volume computing the hair scattering inside 
the hair itself considering it as a cylinder with unit radius.

Starting from the scene representing the hairballs, we created a new scene and started to test
the code there. Then we moved to the pbr models provided by the professor.

# The Code 

We put in yocto_math the constant variables needed for the functions that are shared throughout all
the functions. The pbr code initialized them in the constructor for the BSDF classes, we dediced to choose
to put the globally unique.

We made a distinction in the trace_path function: since the floor and the hairball were treated as 
surfaces as they were the same thing, we had to distinguish the fact that the hair object should be rendered in 
a different way. So we added the "if condition" in which we compute the different code employed to render
the hair objects.

if(object->shape->lines.empty()){
   //handle surfaces
}else{
  //handle hair
}
At every iteration, we compute the "h" value as:

                          h = -1+2*rand1f(rng);

this represents the offset along the curve width where the ray intersect the oriented ribbon.
Since this is a small value, we take it at random clamped between -1 and 1 as the paper suggests.
Once the beam of light touches the hair, the incoming ray must be computed.
The incoming ray can be computed in two different ways. At each round we evaluate it as
either throught the call of the sample_lights function
or throught the call of Sample_f function. The choice can be made at random at each cycle iteration.

The weight vector in trace_path is then updated as follows:

weight *= eval_f(outgoing,incoming) / (0.5f * PDF(outgoing,incoming) +
                      			          0.5f * sample_lights_pdf(scene, position, incoming));

eval_f is used to evaluate the bsdf along the curve of the single hair,
PDF samples the ray according to the bsdf and then we compute the sample of the lights as well.

The function eval_f evaluates the bsdf. Some quantities must be taken into account for doing it
such as the the sine and cosine of the angle θ that each direction makes with the plane perpendicular to the curve, and
the angle φ in the azimuthal coordinate system.

We evaluate the Bsdf as a sum over different terms which regard a particular type 
of light scattering inside the hair, we have:
- light completely reflected from the surface of the hair's cuticle (p=0)
- light is transmitted through the hair and leave the other side(p=1)
- light gets transmitted into the hair and reflected back into it again before being transmitted back out(p=2)
- etc (p=3.. and so forth)

Each term is then summed up into the "fsum" variable, whose type differs with regard to the paper implementation.
The paper uses the Spectrum Type, which represents the incident radiance at the origin of the ray; we decided to
convert it into a vec3f due to yocto library base workspace.
The computation of fsum uses the Azimuthal and Longitudinal components, evaluated through the Np and Mp functions.
Along this two terms, the Absorption component is evaluated through the Ap function that describes how much of the incident light is affected by each of the scattering modes. This particular aspect will be fully covered later in this report.

The Sample_f function samples the direction of the incoming ray. To sample it we need 4 samples, which are taken as random through the DemuxFloat function. Then we have a loop over the pdf values evaluated through the ComputeApPdf and then
decide the term "p" to sample for hair scattering. This will lead to the sampling of wi(incoming ray).

PDF function computes the pdf for sampled hair scattering direction wi. 

# Output  

The key parameters for the output images are represented by the the two different roughness values and the sigma.
The roughness is not unique, since we are considering two planes, we need two roughnesses:
-Azimuthal Roughness (beta_n)
-Longitudinal Rougness (beta_m)
The sigma has been converted into a vec3f, while the paper's implementation treats it as a Spectrum, and represents the absorption term. This is what gives the hair and the fur its color. 
To compute the Transmittance we employ the sigma value:

                         T = std::exp(-sigma*l)

l = total segment length in terms of the hair diameter

The paper presents 3 different values for the sigma, one for blonde hair, one for brown hair and one for black hair.
We reproduced the experiments on the hairballs and on the models provided by the professor and here is the result.

sigma = {0.06,0.1,0.2} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/blonde_paper_floor.jpg)

sigma = {0.84, 1.39, 2.74} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/brown_paper_floor.jpg)

sigma = {3.35, 5.58, 10.96} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/black_paper_floor.jpg)


In all cases we have:
beta_n = 0.3
beta_m = 0.125
Samples ppixel = 1080
Resolution = 1280

However, since the sigma for the blonde hair seems to be too bright in our implementation,
we decided to change it to {0.06,0.14,0.3}, trying to soften the effect of the light on the object.
The two roughness have been changed as well, now we have beta_n=beta_m= 0.1
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/blonde_01_floor.jpg)

We also included the black hairball with different values of roughnesses. In this case,
we employed the same used in the "revisited" blonde hairball.

beta_n = 0.1
beta_m = 0.1
sigma = {0.84, 1.39, 2.74} 
Samples ppixel = 1080
Resolution = 1280
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/brown_01_floor.jpg)


Now, we propose the rendering on the hair models provided by the professor

Natural Hair

sigma = {0.06,0.14,0.3}
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/natural_blonde.jpg)

{0.84, 1.39, 2.74} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/natural_brown.jpg)

{3.35, 5.58, 10.96} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/natural_black.jpg)

In all cases we have:
beta_n = 0.3
beta_m = 0.125
Samples ppixel = 1080
Resolution = 1280

Straight Hair

sigma = {0.06,0.14,0.3}
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/straight_blonde.jpg)

{0.84, 1.39, 2.74} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/straight_brown.jpg)

{3.35, 5.58, 10.96} 
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/straight_black.jpg)


In all cases we have:
beta_n = 0.3
beta_m = 0.125
Samples ppixel = 1080
Resolution = 1280

# Performances 

These are the rendering times for the Hairballs

Time for the Blonde hairball
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/blonde_hairball_performance.png)

Time for the Brown hairball
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/brown_hairball_performance.png)

Time for the Black Hairball
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/black_hairball_performance.png)

As we can see, the black hairball has the shorter rendering time; we think this is due to the high absorption 
coefficient of the black, which turns into having less time in rendering hairs of black color.
The Brown and the Blonde hair have roughly the same time; having a color which is different from the black means to
have high rendering times.

Let's see the rendering times for the Straight Hair

Time for Straight Blonde Hair
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/straight_blonde_performance.png)

Time for Straight Brown Hair
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/straight_brown_performance.png)

Time for Straight Black Hair
![alt_text](https://github.com/matteorusso27/libs/blob/master/Results/straight_black_performance.png)

As discussed earlier, the rendering times for the black are shorter than the others.

Since the times involving the Natural models were roughly the same as the ones for the 
straight hair, we decided not to include them.
We noticed that the rendering times involving the hairball were longer than
the ones with the natural/straight models, this could be due to the different configuration of the models.

# Run the code
Go to libs/yocto_pathtrace/yocto_pathtrace.cpp at line 1172 and choose either the sigma
and/or the roughnesses desired.
Then build

                      bash build.sh

And run the code through 

                    ./bin/yscenetrace tests/three_balls/"SCENE" -o output.jpg -t path -s 1080 -r 1280

Decide to render different "SCENE" by selecting:
-hair_scene
or
-one_ball_with_floor.json
