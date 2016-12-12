---
author:
- 'James G. Malcolm'
biblio-files: '/Users/malcolm/repos/cv/references'
bibliography: '/Users/malcolm/repos/cv/references.bib'
csl: 'ieee.csl'
title: Filtered Tractography
...

-   Background
    -   Overview of Diffusion Imaging
    -   Modeling
        -   Imaging the tissue
        -   Parametric models
        -   Nonparametric models
        -   Regularization
        -   Characterizing tissue
    -   Tractography
        -   Deterministic tractography
        -   Probabilistic tractography
        -   Global tractography
        -   Validation
    -   Applications
        -   Volume segmentation
        -   Fiber clustering
        -   Connectivity
        -   Tissue analysis
    -   Summary
-   Filtered Tractography
    -   Summary
    -   Modeling local fiber orientations
        -   Diffusion tensors
        -   Watson directional function
    -   Estimating the fiber model
    -   The algorithm
-   Results
    -   Tensors
        -   Synthetic tractography
        -   Angular resolution
        -   Estimated quantities
        -   Volume fractions
        -   Three-fiber crossings
        -   <span>*In vivo*</span>tractography
    -   Watson directional functions
        -   Signal reconstruction and angular resolution
        -   *In vivo* tractography
-   Weighted Mixtures
    -   Summary
        -   The weighted model
        -   Constrained estimation
    -   Experiments
        -   Synthetic validation
        -   *In vivo* tractography
    -   Summary
-   Validation
    -   Summary
    -   Results and Discussion
-   Tract-based Statistical Analysis
    -   Summary
    -   Introduction
    -   Our contributions
    -   Method
    -   Tractography and fiber comparison
    -   Results
        -   Synthetic validation
        -   *In vivo* model comparison
    -   Conclusion
-   Summary

Computer vision encompasses a host of computational techniques to
process visual information. Medical imagery is one particular area of
application where data comes in various forms: X-rays, ultrasound
probes, MRI volumes, EEG recordings, NMR spectroscopy, etc. This
dissertation is concerned with techniques for accurate reconstruction of
neural pathways from diffusion magnetic resonance imagery (dMRI).

This dissertation describes a filtered approach to neural tractography.
Existing methods independently estimate the diffusion model at each
voxel so there is no running knowledge of confidence in the estimation
process. We propose using tractography to drive estimation of the local
diffusion model. Toward this end, we formulate fiber tracking as
recursive estimation: at each step of tracing the fiber, the current
estimate is guided by those previous.

We argue that this approach is more accurate than conventional
techniques. Experiments demonstrate that this filtered approach
significantly improves the angular resolution at crossings and
branchings. Further, we confirm its ability to trace through regions
known to contain such crossing and branching while providing inherent
path regularization.

We also argue that this approach is flexible. Experiments demonstrate
using various models in the estimation process, specifically
combinations of Watson directional functions and rank-2 tensors.
Further, this dissertation includes an extension of the technique to
weighted mixtures using a constrained filter.

Background
==========

Overview of Diffusion Imaging
-----------------------------

The advent of diffusion magnetic resonance imaging (dMRI) has provided
the opportunity for non-invasive investigation of neural architecture.
While structural MRI has long been used to image soft tissue and bone,
dMRI provides additional insight into tissue microstructure by measuring
its microscopic diffusion characteristics. To accomplish this, the
magnetic field induces the movement of water while the presence of cell
membranes, fibers, or other macromolecules hinder this movement. By
varying the direction and strength of the magnetic fields, we
essentially use the water molecules as a probe to get a sense of the
local tissue structure.

At the lowest level this diffusion pattern provides several insights.
For example, in fibrous tissue the dominant direction of allowed
diffusion corresponds the underlying direction of fibers. In addition,
quantifying the anisotropy of the diffusion pattern can also provide
useful biomarkers. Several models have been proposed to interpret
scanner measurements, ranging from geometric abstractions to those with
biological motivation. In we will introduce various models and methods
for interpreting the diffusion measurements.

By connecting these local orientation models, tractography attempts to
reconstruct the neural pathways. Tracing out these pathways, we begin to
see how neurons originating from one region connect to other regions and
how well-defined those connections may be. Not only can we examine
properties of the local tissue but we begin to see the global functional
architecture of the brain, but for such studies, the quality of the
results relies heavily on the chosen fiber representation and the method
of reconstructing pathways. In we will describe several techniques for
tracing out pathways.

![Cutaway showing tractography throughout the left hemisphere colored by
FA to indicate diffusion strength @Malcolm2009ipmi. From this view, the
fornix and cingulum bundle are visible near the
center.](case01045_2T_half_fa)

[fig:rh~f~a]

At the highest level, neuroscientists can use the results of local
modeling and tractography to examine individuals or groups of
individuals. In we will survey approaches to segment tissue with
boundaries indistinguishable with structural MRI, apply network analysis
to characterize the macroscopic neural architecture, reconstruct fiber
bundles from individual fiber traces, and analyze groups of individuals.

Modeling {#sec:modeling}
--------

### Imaging the tissue

The overall signal observed in an dMRI image voxel (millimetric) is the
superposition of signals from many underlying molecules probing the
tissue (micrometric). Thus, the image contrast is related to the
strength of water diffusion. At each image voxel, diffusion is measured
along a set of distinct gradients,
${{\ensuremath{\mathbf u}\xspace}}_1,...,{{\ensuremath{\mathbf u}\xspace}}_n\in{\ensuremath{\mathbb R}}^3$,
producing the corresponding signal,
${{\ensuremath{\mathbf s}\xspace}}={\ensuremath{[\, s_1,...,s_n \,]}\xspace}^T\in{\ensuremath{\mathbb R}}^n$.
A general weighted formulation that relates the measured diffusion
signal to the underlying fiber architecture may be written as,

$$\label{eq:general_model}
  s_i = s_0 \sum_j w_j e^{ -b_j {{\ensuremath{\mathbf u}\xspace}}_i^T D_j {{\ensuremath{\mathbf u}\xspace}}_i },$$

where $s_0$ is a baseline signal intensity, $b_j$ is the b-value, an
acquisition-specific constant, $w_j$ are convex weights, and $D_j$ is a
tensor describing a diffusion pattern. One of the first acquisition
schemes developed, diffusion tensor imaging (DTI) utilizes these
measurements to compute a Gaussian estimate of the diffusion orientation
and strength at each voxel @Basser1994jmr [@LeBihan2003].

Going beyond this macroscopic description of diffusion, various higher
resolution acquisition techniques have been developed to capture more
information about the diffusion pattern. One of the first techniques,
Diffusion Spectrum Imaging (DSI) measures the diffusion process at
various scales by sampling densely throughout the voxel @Wedeen2005
[@Hagmann2005phd]. From this, the Fourier transform is used to convert
the signal to a diffusion probability distribution. Due to the large
number of samples acquired (usually more than 256 gradient directions),
this scheme provides a much more accurate description of the diffusion
process. However, on account of the large acquisition time (of the order
of 1-2 hours per subject), this technique is not typically used in
clinical scans and its use is restricted to few research applications.

Instead of spatially sampling the diffusion in a lattice throughout the
voxel, a spherical shell sampling could be used @Tuch2002phd. Using this
sampling technique, the authors in @Tuch2004 demonstrated that the shape
of the diffusion probability could be recovered from the images acquired
on a single spherical shell. This significantly reduced the acquisition
time, while providing most of the information about the underlying
diffusion in the tissue. This naturally led to the application of
techniques for estimating functions on a spherical domain. For example,
Q-ball imaging (QBI) demonstrated a spherical version of the Fourier
transform to reconstruct the probability diffusion as an isosurface
@Tuch2002phd.

To begin studying the microstructure of fibers with these imaging
techniques, we need models to interpret these diffusion measurements.
Such models fall broadly into two categories: parametric and
nonparametric.

### Parametric models

One of the simplest models of diffusion is a Gaussian distribution: an
elliptic (anisotropic) shape indicating a strong diffusion direction
while a more rounded surface (isotropic) indicating less certainty in
any particular direction (see ). While robust, assuming this Gaussian
model is inadequate in cases of mixed fiber presence or more complex
orientations where the signal may indicate a non-Gaussian pattern
@Alexander2002 [@Frank2002; @Tuch2002]. To handle these complex
patterns, higher resolution imaging and more flexible parametric models
have been proposed including mixtures of tensors @Alexander2001
[@Tuch2002; @Behrens2007; @Hosey2005; @Parker2005; @Kreher2005; @Peled2006]
and directional functions @McGraw2006 [@Kaden2007; @Rathi2009mia_w].
While these typically require the number of components to be fixed or
estimated separately, more continuous mixtures have also been proposed
@Jian2007ni. Further, biologically inspired models and tailored
acquisition schemes have been proposed to estimate physical tissue
microstructure @Assaf2005 [@Assaf2008]; see @Alexander2009 for more.

### Nonparametric models

Nonparametric models can often provide more information about the
diffusion pattern. Instead of modeling a discrete number of fibers as in
parametric models, nonparametric techniques estimate a spherical
orientation distribution function indicating potential fiber directions
and the relative certainty thereof. For this estimation, an assortment
of surface reconstruction methods have been introduced: Q-ball imaging
to directly transform the signal into a probability surface @Tuch2004,
spherical harmonic representations @Frank2002
[@Anderson2005; @Hess2006; @Descoteaux2007mrm], higher-order tensors
@Ozarslan2003 [@Basser2007; @Barmpoutis2009], diffusion profile
transforms @Jansons2003 [@Ozarslan2006], deconvolution with an assumed
single-fiber signal response @Tournier2004 [@Jian2007dot], and more.
shows a spherical harmonic reconstruction of the signal. Compare this to
the original signal in .

It is important to keep in mind that there is a often distinction made
between the reconstructed diffusion orientation distribution function
and the putative fiber orientation distribution function; while most
techniques estimate the diffusion function, its relation to the
underlying fiber function is still an open problem. Spherical
convolution is designed to directly transform the signal into a fiber
distribution @Alexander2005ipmi
[@Jansons2003; @Anderson2005; @Tournier2004], yet diffusion sharpening
strategies have been developed to deal with Q-ball and diffusion
functions @Descoteaux2009tmi.

While parametric methods directly describe the principal diffusion
directions, interpreting the diffusion pattern from model independent
representations typically involves determining the number and
orientation of principal diffusion directions present. A common
technique is to find them as surface maxima of the diffusion function
@Hess2006 [@Tournier2004; @Bloy2008; @Descoteaux2009tmi], while another
approach is to decompose a high-order tensor representation of the
diffusion function into a mixture of rank-1 tensors @Schultz2008.

### Regularization

As in all physical systems, the measurement noise plays a nontrivial
role, and so several techniques have been proposed to regularize the
estimation. One could start by directly regularizing the MRI signal by
designing filters based on the various signal noise models @Koay2006
[@Assemlal2008; @AjaFernandez2008]. Alternatively, one could estimate
the diffusion tensor field and then correct these estimated quantities
@Tschumperle2002. For spherical harmonic modeling, a regularization term
can be been directly included in the least squares formulation @Hess2006
[@Descoteaux2007mrm]. Attempts such as these to manipulate diffusion
weighted images or tensor fields have received considerable attention
regarding appropriate algebraic and numeric treatments @Tschumperle2002
[@Batchelor2005; @Fletcher2007riem; @Kindlmann2007miccai].

Instead of regularizing signal or model parameters directly, an
alternative approach is to infer the underlying geometry of the vector
field @Savadjiev2006. Another interesting approach treats each newly
acquired diffusion image as a new system measurement. Since diffusion
tensors and spherical harmonics can be estimated within a least-squares
framework, one can use a Kalman filter to update the estimate and
optionally stop the scan when the model parameters converge @Poupon2008.
Further, this online technique can be used to alter the gradient set so
that, were the scan to be stopped early, the gradients up to that point
are optimally spread (active imaging) @Deriche2009.

### Characterizing tissue

The goal of diffusion imaging is to draw inferences from the diffusion
measurements. As a starting point, one often converts the diffusion
weighted image volumes to a scalar volume much like structural MRI or CT
images. Starting with the standard Gaussian diffusion tensor model, an
assortment of scalar measures have been proposed to quantify the size,
orientation, and shape of the diffusion pattern @Basser1996
[@Westin2002]. For example, fractional anisotropy quantifies the
deviation from an isotropic tensor, an appealing quantity because it
corresponds to the strength of diffusion while remaining invariant to
orientation. Derivatives of these scalar measures have also been
proposed to capture more information about the local neighborhood
@Kindlmann2007tmi [@Savadjiev2009], and these measures have been
extended to high-order tensors @Ozarslan2005. Further, a definition of
generalized anisotropy has been proposed to directly characterize
anisotropy in terms of variance in the signal, hence avoiding an assumed
model @Tuch2002. While geometric in nature, studies have shown these to
be reasonable proxy measures for neural myelination @Norris2001
[@Beaulieu2002; @Jones2003]. Some studies have also examined the
sensitivity of such measures against image acquisition schemes
@Whitcher2008 [@Chung2006].

Meaningful visualization of diffusion images is difficult because of
their multivariate nature, and much is lost when reducing the spectral
signal down to scalar intensity volumes. Several geometric abstractions
have been proposed to convey more information. Since the most common
voxel model is still the Gaussian diffusion tensor, most of the effort
has focused on visualizing this basic element. The most common glyph is
an ellipsoid simultaneously representing the size, shape, and
orientation; however, since tensors have six free parameters, more
elaborate representations have been proposed to visualize these
additional dimensions using color, shading, or subtle variations in
shape @Westin2002 [@Ennis2005; @Vilanova2006; @Kindlmann2007miccai].
Apart from tensors, visualization strategies for other models have
received comparatively little attention, the typical approach being to
simply to visualize the diffusion isosurface at each voxel.

1em A vast literature exists on methods of acquisition, modeling,
reconstruction, and visualization of diffusion images. For a
comprehensive view, we suggest @Tuch2002phd
[@Minati2007; @Hagmann2005phd; @Westin2002; @Alexander2005; @Descoteaux2008].

![Tractography from the center of the corpus callosum (seed region in
yellow). The single-tensor model (left) captures only the corona radiata
and misses the lateral pathways known to exist. The two-tensor method
@Malcolm2009ipmi (right) reveals many of these missing pathways
(highlighted in blue). (From @Malcolm2009ipmi)](case01045_1T_tc)

![Tractography from the center of the corpus callosum (seed region in
yellow). The single-tensor model (left) captures only the corona radiata
and misses the lateral pathways known to exist. The two-tensor method
@Malcolm2009ipmi (right) reveals many of these missing pathways
(highlighted in blue). (From @Malcolm2009ipmi)](case01045_2T_tc)

[fig:filtered]

Tractography {#sec:tractography}
------------

To compliment the wide assortment of techniques for signal modeling and
reconstruction, there is an equally wide range of techniques to infer
neural pathways. At the local level, one may categorize them either as
tracing individual connections between regions or as diffusing out to
estimate the probability of connection between regions. In addition,
more global approaches have been developed to consider, not just the
local orientations, but the suitability of entire paths when inferring
connections.

### Deterministic tractography

Deterministic tractography involves directly following the diffusion
pathways. Typically, one places several starting points (seeds) in one
region of interest and iteratively traces from one voxel to the next,
essentially path integration in a vector field. One terminates these
fiber bundles when the local diffusion appears week or upon reaching a
target region. offers a glimpse from inside the brain using this basic
approach. Often additional regions are used as masks to post-process
results, <span>*e.g*</span>pathways from region A but not touching
region B.

In the single tensor model, standard streamline tractography follows the
principal diffusion direction of the tensor @Mori2002, while multi-fiber
models often include techniques for determining the number of fibers
present or when pathways branch @Hagmann2004 [@Kreher2005; @Guo2006].
Since individual voxel measurements may be unreliable, several
techniques have been developed for regularization. For example, using
the estimate from the previous position @Lazar2003 [@Zhukov2002] as well
as filtering formulations for path regularization @Gossl2002 and
model-based estimation @Malcolm2009ipmi. The choice of model and
optimization mechanism can drastically effect the final tracts. To
illustrate, shows tractography from the center of the corpus callosum
using a single-tensor model and a two-tensor model using the filtered
technique from @Malcolm2009ipmi.

### Probabilistic tractography

While discrete paths intuitively represent the putative fiber pathways
of interest, they tend to ignore the inherent uncertainty in estimating
the principle diffusion directions in each voxel. Instead of tracing
discrete paths to connect voxels, one may instead query the probability
of voxel-to-voxel connections given the diffusion probability
distributions reconstructed in each voxel.

Several approaches have been developed based on sampling. For example,
one might run streamline tensor tractography treating each as a Monte
Carlo sample; the more particles that take a particular path, the more
likely that particular fiber pathway @Koch2002
[@Behrens2003; @Parker2003; @Bjornemo2002]. Another approach would be to
consider more of the continuous diffusion field from Q-ball or other
reconstructions @Tuch2000
[@Batchelor2001; @Perrin2005; @Parker2005; @Friman2006; @Zhang2009]. By
making high curvature paths unlikely, path regularization can be
naturally enforced within the probabilistic framework. Another approach
is to propagate an arrival-time isosurface from the seed region out
through the diffusion field, the front evolution force being a function
of the local diffusivity @Batchelor2001 [@Campbell2005; @Tournier2003].

Using the full diffusion reconstruction to guide particle diffusion has
the advantage of naturally handling uncertainty in diffusion
measurements, but for that same reason it tends toward diffuse
tractography and false-positive connections. One option is to constrain
diffusivity by fitting a model, thereby ensuring definite diffusion
directions yet still taking into account some uncertainty @Parker2005
[@Friman2006; @Behrens2003]. A direct extension is to introduce a model
selection mechanism to allow for additional components where appropriate
@Behrens2007 [@Freidlin2007]. However, one could stay with the
nonparametric representations and instead sharpen the diffusion profile
to draw out the underlying fiber orientations @Tournier2007
[@Descoteaux2009tmi].

### Global tractography

Despite advances in voxel modeling, discerning the underlying fiber
configuration has proven difficult. For example, looking at a single
voxel, the symmetry inherent in the diffusion measurements makes it
difficult to tell if the observed pattern represents a fiber curving
through the voxel or instead represents a fanning pattern. Reliable and
accurate fiber resolution requires more information than that of a
single voxel. For example, instead of estimating the fiber orientation,
one could instead infer the geometry of the entire neighborhood
@Savadjiev2008.

Going a step further, one could say that reliable and accurate
connectivity resolution requires even more information, beyond simply a
voxel neighborhood. In some respects, probabilistic tractography can be
seen to take into account more global information. By spawning thousands
of particles, each attempting to form an individual connection,
probabilistic techniques are able to explore more possibilities before
picking those that are likely @Parker2003. However, if these particles
still only look at the local signal as they propagate from one voxel to
the next, then they remain susceptible to local regions of uncertainty.
Even those with resampling schemes are susceptible since the final
result is still a product of the method used in local tracing
@Bjornemo2002 [@Zhang2009].

A natural step to address such problems is to introduce global
connectivity information into local optimization procedures of
techniques mentioned above. The work of @Jbabdi2007 does this by
extending the local Bayesian formulation in @Behrens2007 with an
additional prior that draws upon global connectivity information in
regions of uncertainty. Similarly, one could use an energetic
formulation still with data likelihood and prior terms, but additionally
introduce terms governing the number of components present @Fillard2009.

Another approach is to treat the entire path as the parameter to be
optimized and use global optimization schemes. For example, one could
model pathways as piecewise linear with a data likelihood term based on
signal fit and a prior on spatial coherence of those linear components
@Kreher2008 [@Reisert2009]. One advantage of this path-based approach is
that it somewhat obviates the need for a multi-fiber voxel model;
however, such a flexible global model dramatically increases the
computational burden.

An alternative formulation is to find geodesic paths through the volume.
Again using some form of data likelihood term, such methods then employ
techniques for front propagation to find globally optimal paths of
connection @ODonnell2002
[@Pichon2005; @Prados2006; @Lenglet2004eccv; @Fletcher2007ipmi].

Tractography is often used in group studies which typically require a
common atlas for inter-subject comparison. Beginning with the end in
mind, one could determine a reference bundle as a template and use this
to drive tractography. This naturally ensures both the general geometric
form of the solution and a direct correspondence between subjects
@Eckstein2009 [@Goodlett2009]. Alternatively, the tract seeding and
other algorithm parameters could be optimized until the tracts (data
driven) approach the reference (data prior) @Clayden2007. Since this
requires pre-specifying such a reference bundle, information that may be
unavailable or difficult to obtain, one could even incorporate the
formulation of the reference bundle into the optimization procedure
itself @Clayden2009.

### Validation

In attempting to reconstruct neural pathways virtually, it is important
to keep in mind the inherent uncertainty in such reconstructions. The
resolution of dMRI scanners is at the level of 1-3mm while physical
fiber axons are often an order of magnitude smaller in diameter–a
relationship which leaves much room for error. Some noise or a complex
fiber configuration could simply look like a diffuse signal and cause
probabilistic tractography to stop in its tracks, while a few inaccurate
voxel estimations could easily send the deterministic tracing off course
to produce a false-positive connection. Even global methods could
produce a tract that fits the signal quite well but incidentally jumps
over an actual boundary in one or two voxels it thinks are noise.
Consequently, a common question is: Are these pathways really present?

With this in mind, an active area of study is validating such results.
Since physical dissection often requires weeks of tedious effort, many
techniques have been used for validating these virtual dissections. A
common starting point is to employ synthetic and physical phantoms with
known parameters when evaluating new methods @Poupon2008phantom. When
possible, imaging before and after injecting radio-opaque dyes directly
into the tissue can provide some of the best evidence for comparison
@Lin2003 [@Dauguet2007]. Another powerful approach is to apply bootstrap
sampling or other non-parametric statistical tests to judge the
sensitivity and reproducibility of resulting tractography against
algorithm parameters, image acquisition, and even signal noise
@Lazar2005
[@Jones2005; @Gigandet2008; @Whitcher2008; @Chung2006; @Clayden2007].

Applications {#sec:applications}
------------

Having outlined various models and methods of reconstructing pathways,
we now briefly cover several methods of further analysis.

### Volume segmentation

Medical image segmentation has a long history, much of it focused on
scalar intensity-based segmentation of anatomy. For neural segmentation,
structural MRI easily reveals the boundaries between gray-matter and
white-matter, and anatomic priors have helped further segment some
internal structures @Pohl2007ipmi; however, the boundaries between many
structures in the brain are remain invisible with structural MRI alone.
The introduction of dMRI has provided new discriminating evidence in
such cases where tissue may appear homogeneous on structural MRI or CT
but contain distinct fiber populations.

To begin, most work has focused segmentation of the estimated tensor
fields. Using suitable metrics to compare tensors, these techniques
often borrow directly from active contour or graph cut segmentation with
the approach of separating distributions. For example, one could define
a Gaussian distribution of tensors to approximate a structure of
interest @Rousson2004 [@deLuis-Garcia2007]. For tissues with more
heterogeneous fiber populations, <span>*e.g*</span>the corpus callosum
as it bends, such global parametric representations are unsuitable. For
this, nonparametric approaches are more appropriate at capturing the
variation throughout such structures @Rathi2007 [@Malcolm2007tc].
Another approach to capture such variation is to limit the parametric
distributions to local regions of support, essentially robust edge
detection @Lankton2008mmbia.

In we see a graph cut segmentation of the corpus callosum
@Malcolm2007tc. The color-coded fractional anisotropy image is shown for
visualization while segmentation was performed on the underlying tensor
data. If statistics are computing ignoring the tensor manifold
(Euclidean assumption), the final segmentation fails . If statistics are
computing via a Riemannian mapping that respects this structure, the
final segmentation is accurate . This highlights the need for
appropriate algebraic treatment of tensors and other non-Euclidean
models.

[fig:corpus]

An altogether different approach to segmenting a structure is to divide
it up according to where portions connect elsewhere. For example, the
thalamus contains several nuclei indistinguishable in standard MR or
even with contrast. After tracing connections from the thalamus to the
cortex, one study demonstrated that grouping these connections revealed
the underlying nuclei @Behrens2003nn.

### Fiber clustering

The raw output of full-brain tractography can produce hundreds of
thousands of such tracings, an overwhelming amount of information. One
approach to understanding and visualizing such results is to group
individual tracings into fiber bundles. Such techniques are typically
based around two important design choices: the method of comparing
fibers, and the method of clustering those fibers.

In comparing two fibers, one often starts by defining a distance
measure, these typically being based on some point-to-point
correspondence between the fibers @Ding2003
[@Corouge2006; @ODonnell2007tmi]. With this correspondence in hand, one
of the most common distances is then to take the mean closest point
distance between the two fibers (Hausdorff distance). An alternative is
to transform each fiber to a new vector space with a natural norm,
<span>*e.g*</span>a fiber of any length can be encoded with only the
mean and covariance of points along its path and then use the L$_2$
distance @Brun2004. An altogether different approach is to consider the
spatial overlap between fibers @Wang2009 [@Wasserman2009]. Since
full-brain tractography often contains many small broken fragments as it
tries to trace out bundles, such fragments are often separated from
their actual cluster. Measures of spatial overlap may be more robust in
such cases. In each of these methods, fibers were only considered as
sequences of points, <span>*i.e*</span>connections and orientations were
ignored. Recent work demonstrates that incorporating such considerations
provides robust descriptors of fiber bundles @Durrelman2009.

![Full-brain streamline tractography clustered using affinity
propagation @Malcolm2009cukf_ext. Viewed from the outside (left) and
inside cutting away the left hemisphere (right). Among the visible
structures, we see the cingulum bundle (yellow), internal capsule (red),
and arcuate (purple).](case01045_1t_rh_ext "fig:") ![Full-brain
streamline tractography clustered using affinity propagation
@Malcolm2009cukf_ext. Viewed from the outside (left) and inside cutting
away the left hemisphere (right). Among the visible structures, we see
the cingulum bundle (yellow), internal capsule (red), and arcuate
(purple).](case01045_1t_rh_int "fig:")

[fig:rh]

Based on these distances, several methods have been developed to cluster
the fibers. Spectral methods typically begin with the construction of a
Gram matrix encoding the pairwise affinity between any two fibers. After
which normalized cuts can be applied to partition the Gram matrix and
hence the fibers @Brun2004. Affinity propagation has recently been
demonstrated as an efficient and robust alternative which automatically
determines the number of clusters to support a specified cluster size
preference @Leemans2009 [@Malcolm2009cukf_ext]. In shows how clustering
can automatically reveal known structures and provide a more coherent
view of the brain. In addition, clustering can be used to judge
outliers. For example, reveals several streamlines that appear to have
gone off track relative to the cluster centers.

[fig:fo]

Another clustering approach is to use the inner product space itself.
For example, one can efficiently group directly on the induced manifold
by iteratively joining fibers most similar until the desired clustering
emerges @Wasserman2009. To avoid construction of the large Gram matrix,
variants of expectation maximization have been demonstrated to
iteratively cluster fibers, an approach that naturally lends itself to
incorporating anatomic priors @Wang2009
[@ODonnell2007tmi; @Maddah2008media]. Alternatively, one can begin with
the end in mind by registering a reference fiber bundle template to
patients thus obviating any need for later spatial normalization or
correspondence @Clayden2009.

### Connectivity

While tissue segmentation can provide global cues of neural
organization, it tells little of the contribution of individual
elements. Similarly, while clustered tracings are easily visualized,
deciphering the flood of information from full-brain tractography
demands more comprehensive quantitative analysis. For this, much has
been borrowed from network analysis to characterize the neural topology.
To start, instead of segmenting fibers into bundles, one can begin by
classifying voxels into hubs or subregions into subnetworks @Sporns2007
[@Gong2009; @Hagmann2007; @Hagmann2008].

Dividing the brain up into major functional hubs, one can then view it
as a graphical network as in . Each of these edges is then often
weighted as a function of connection strength @Hagmann2007, but may also
incorporate functional correlation to give further evidence of
connectivity.

![The brain viewed as a network of weighted connections. Each edge
represents a possible connection and is weighted by the strength of that
path. Many techniques from network analysis are applied to reveal hubs
and subnetworks within this macroscopic
view.](case01045_connections_320)

[fig:connect]

One of the first results of such analysis was the discovery of dense
hubs linked by short pathways, a characteristic observed in many complex
physical systems (small-world phenomena). Another interesting finding
came from combining anatomic connections from dMRI with neuronal
activity provided by fMRI @Honey2009. They found that areas which are
functionally connected are often not structurally connected, hence
tractography alone does not provide the entire picture.

For a recent review of this emerging field of structural and functional
network analysis, we recommend @Bullmore2009.

### Tissue analysis

![Plotting FA as a function of arc-length to examine local fluctuations.
Fibers are selected that connect the left and right seed regions
(green). Notice how the FA from single-tensor (blue) is lower in regions
of crossing compared to two-tensor FA (red). (From
@Malcolm2009study)](case01026_03_ds)

[fig:fa~d~s]

In forming population studies, there are several approaches for framing
the analysis among patients. For example, voxel-based studies examine
tissue characteristics in regions of interest @Ashburner2000.
Discriminant analysis has been applied to determine such regions
@Caan2006. Alternatively, one could also perform regression on the full
image volume taking into account not only variation in diffusion but
also in the full anatomy @Rohlfing2009. In contrast, tract-based studies
incorporate the results of tractography to use fiber pathways as the
frame of reference @Ding2003 [@Smith2006], and several studies have
demonstrated the importance of taking into account local fluctuations in
estimated diffusion @Corouge2006
[@ODonnell2009; @Maddah2008media; @Goodlett2009; @Yushkevich2008].

A common approach in many of these studies is to focus on characterizing
individual pathways or bundles. To illustrate this analysis, shows
fibers connecting a small region in each hemisphere. We then average FA
plotted along the bundle as a function of arc-length. Further, we plot
the FA from both single- and two-tensor models to show how different
models often produce very different tissue properties @Malcolm2009study.

Several reviews exist documenting the application and findings of using
various methods @Lim2002 [@Horsfield2002; @Kubicki2007].

Summary
-------

Diffusion MRI has provided an unprecedented view of neural architecture.
With each year, we develop better image acquisition schemes, more
appropriate diffusion models, more accurate pathway reconstruction, and
more sensitive analysis.

In this survey, we began with an overview of the various imaging
techniques and diffusion models. While many acquisition sequences have
become widely distributed for high angular resolution imaging, work
continues in developing sequences and models capable of accurate
resolution of biological properties such as axon diameter and degree of
myelination @Assaf2008 [@Alexander2009]. We then reviewed various
parametric models starting with the diffusion tensor on up to various
mixture models as well as high-order tensors. Work continues to develop
more accurate and reliable model estimation by incorporating information
from neighboring voxels @Savadjiev2008 [@Malcolm2009ipmi]. Further,
scalar measures derived from these models similarly benefit from
incorporating neighborhood information @Savadjiev2009.

Next we outlined various methods of tractography to infer connectivity.
Broadly, these techniques took either a deterministic or probabilistic
approach. We also documented the recent trend toward global approaches,
those that combine local voxel-to-voxel tracing with a sense of the full
path @Fillard2009 [@Reisert2009]. Even with such considerations,
tractography is has proven quite sensitive to image acquisition and
initial conditions, so much work has gone into validation. Common
techniques are the use of physical phantoms @Poupon2008phantom or
statistical tests like bootstrap analysis @Lazar2005
[@Jones2005; @Clayden2007].

Finally, we briefly introduced several machine learning approaches to
make sense of the information found in diffusion imagery. Starting with
segmentation, several techniques for scalar intensity segmentation have
been extended to dMRI. With the advent of full-brain tractography
providing hundreds of thousands of fiber paths, the need to cluster
connections into bundles has become increasingly important. The
application of network analysis to connectivity appears to be an
emerging area of research, especially in combination with alternate
imaging modalities @Bullmore2009. Finally, we noted several approaches
to the analysis of neural tissue itself in regions of interest or along
pathways.

Filtered Tractography
=====================

Summary
-------

Nearly all approaches to tractography fit the model at each voxel
independent of other voxels; however, tractography is a causal process:
we arrive at each new position along the fiber based upon the diffusion
found at the previous position.

We propose to we treat model estimation and tractography as such by
placing this process within a causal filter. As we examine the signal at
each new position, the filter recursively updates the underlying local
model parameters, provides the variance of that estimate, and indicates
the direction in which to propagate tractography. provides an overview
of this recursive process.

[fig:overview]

To begin estimating within a finite dimensional filter, we model the
diffusion signal using a mixture of parametric directional functions. We
choose parametric models since they provide a compact representation of
the signal parameterized by the principal diffusion direction and
scaling parameters influencing anisotropy, and further allows analytic
reconstruction of the oriented diffusion function from those parameters
@malcolm2010tmi [@Rathi2009mia_w]. This enables estimation directly from
the raw signal without separate preprocessing or regularization. Because
the signal reconstruction is nonlinear, we use the unscented Kalman
filter to estimate the model parameters and then propagate in the most
consistent direction. Using causal estimation in this way yields
inherent path regularization and accurate fiber resolution at crossing
angles not found with independent optimization. In a loop, the filter
estimates the model at the current position, moves a step in the most
consistent direction, and then begins estimation again. Since each
iteration begins with a near-optimal solution provided by the previous
estimation, the convergence of model fitting is improved and many local
minima are naturally avoided. This approach generalizes to arbitrary
fiber model with finite dimensional parameter space. The bulk of this
dissertation is found among @malcolm2010tmi
[@Malcolm2009ipmi; @malcolm2009cukf].

Several studies have specific relevance to this present work because of
their use of a filtering strategy in either orientation estimation or
tractography. In extending standard streamline tractography to enhance
path regularization, @Gossl2002 move curve integration into a linear
Kalman filter while @Zhukov2002 incorporate a moving least squares
estimator. Alternatively, one could use a particle filter to place a
prior on the direction of propagation @Zhang2009. Since these methods
model only the position of the fiber, not the local fiber model, they
are inherently focused on path regularization rather than estimating the
underlying fiber structure. Finally, @Poupon2008 proposed using a linear
Kalman filter for online, direct estimation of either single-tensor or
harmonic coefficients while successive diffusion image slices are
acquired, while @Deriche2009 revisited the technique to account for
proper regularization and proposed a method to quickly determine optimal
gradient set orderings.

provides the necessary background on modeling the measurement signal
using directional functions and defines the specific fiber models
employed in this study. Then, describes how this model may be estimated
using an unscented Kalman filter.

Modeling local fiber orientations {#sec:model}
---------------------------------

[fig:sweetness]

In diffusion weighted imaging, image contrast is related to the strength
of water diffusion, and our goal is to accurately relate these signals
to an underlying model of fiber orientation. At each image voxel,
diffusion is measured along a set of distinct gradients,
${{\ensuremath{\mathbf u}\xspace}}_1,...,{{\ensuremath{\mathbf u}\xspace}}_n\in{\ensuremath{\mathbb S}}^2$
(on the unit sphere), producing the corresponding signal,
${{\ensuremath{\mathbf s}\xspace}}={\ensuremath{[\, s_1,...,s_n \,]}\xspace}^T\in{\ensuremath{\mathbb R}}^n$.
For voxels containing a mixed diffusion pattern, a general weighted
formulation may be written as,

$$s_i = s_0 \sum_j w_j e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_j {{\ensuremath{\mathbf u}\xspace}}_i },$$

where $s_0$ is a baseline signal intensity, $b$ is an
acquisition-specific constant, $w_j$ are convex weights, and $D_j$ is a
tensor matrix describing a diffusion pattern.

We now provide definition to the two primary models employed in this
work: equally weighted diffusion tensors @Malcolm2009ipmi
[@malcolm2010tmi] and Watson directional functions@Rathi2009mia_w. Later
in , we return to extend the method to weighted mixtures of diffusion
tensors which requires a different filtering scheme @malcolm2009cukf.

### Diffusion tensors {#sec:equal}

From the general mixture model above (), we choose to start with two
components. This choice is guided by several previous studies which
found two-fiber models to be sufficient at
<span><span>$b\!=\!1000$</span></span>@Tuch2002
[@Kreher2005; @Guo2006; @Zhan2006; @Peled2006; @Behrens2007]. Also, we
assume the shape of each tensor to be ellipsoidal,
<span>*i.e*</span>there is one dominant principal diffusion direction
<span><span>$\mathbf m$</span></span>with eigenvalue
<span><span>$\lambda_1$</span> </span>and the remaining orthonormal
directions have equal eigenvalues
${{\ensuremath{\lambda_2}} \xspace}=\lambda_3$ (as in @Parker2005
[@Friman2006; @Peled2006; @Kaden2007]). Last, as in the study of
@Zhan2006, we fix the weights so that each component contributes
equally. While assuming equally-weighted compartments may limit
flexibility, we found that the eigenvalues adjust to fit the signal in
much the same way a fully weighted model would adjust. We will revisit
this in the experiments and discussion ().

These assumptions then leave us with the following model used in this
study:

$$\label{eq:2T_model}
  s_i = \tfrac{s_0}{2} e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_1 {{\ensuremath{\mathbf u}\xspace}}_i } + \tfrac{s_0}{2} e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_2 {{\ensuremath{\mathbf u}\xspace}}_i } ,$$

where $D_1,D_2$ are each expressible as,
$ D = {{\ensuremath{\lambda_1}} \xspace}{{\ensuremath{\mathbf m}\xspace}}{{\ensuremath{\mathbf m}\xspace}}^T + {{\ensuremath{\lambda_2}} \xspace}\left({\ensuremath{\mathbf p}\xspace}{\ensuremath{\mathbf p}\xspace}^T + {\ensuremath{\mathbf q}\xspace}{\ensuremath{\mathbf q}\xspace}^T\right), $
with
${{\ensuremath{\mathbf m}\xspace}},{\ensuremath{\mathbf p}\xspace},{\ensuremath{\mathbf q}\xspace} \in {\ensuremath{\mathbb S}}^2$
forming an orthonormal basis aligned to the principal diffusion
direction <span><span>$\mathbf m$</span></span>. The free model
parameters are then ${{\ensuremath{\mathbf m}\xspace}}_1$,
${{\ensuremath{\lambda_1}} \xspace}_1$,
${{\ensuremath{\lambda_2}} \xspace}_1$,
${{\ensuremath{\mathbf m}\xspace}}_2$,
${{\ensuremath{\lambda_1}} \xspace}_2$, and
${{\ensuremath{\lambda_2}} \xspace}_2$. In our current implementation,
we restrict each $\lambda$ to be positive. Extending off the two-tensor
model, we can directly formulate a three-tensor version:

$$\label{eq:3T_model}
  s_i = \frac{s_0}{3} \sum_{j=1}^3 e^{-b {{\ensuremath{\mathbf u}\xspace}}_i^T D_j {{\ensuremath{\mathbf u}\xspace}}_i } ,$$

with the additional parameters ${{\ensuremath{\mathbf m}\xspace}}_3$,
${{\ensuremath{\lambda_1}} \xspace}_3$, and
${{\ensuremath{\lambda_2}} \xspace}_3$.

### Watson directional function {#sec:watson}

Considering a single tensor, we now follow the formulation of Rathi
<span>et al</span>@Rathi2009mia_w to define the Watson directional
function which approximates the apparent diffusion pattern. We begin by
noting that any diffusion tensor $D$ can be decomposed as
$D = U \Lambda U^T$, where $U$ is a rotation matrix and $\Lambda$ is a
diagonal matrix with eigenvalues $\{\lambda_1, \lambda_2, \lambda_3\}$.
These eigenvalues determine the shape of the tensor: ellipsoidal,
planar, and spherical. For example, if
$\lambda_1 > \lambda_2 > \lambda_3$, then the shape is ellipsoidal with
the major axis of the ellipsoid pointing to the eigenvector
corresponding to $\lambda_1$. Intuitively, it represents strong
diffusion along that particular direction. When
$\lambda_1 = \lambda_2 > \lambda_3$, the shape is planar indicating
diffusion along orthogonal directions, and finally, when
$\lambda_1 = \lambda_2 = \lambda_3$, the diffusion is spherical
(isotropic).

The most common of these configurations is ellipsoidal with principal
diffusion direction ${{\ensuremath{\mathbf m}\xspace}}$ and eigenvalue
$\lambda_1$, and hence the first step in introducing directional
functions is to approximate the tensor by its first eigenvector
expansion:
$D \approx \lambda_1 {{\ensuremath{\mathbf m}\xspace}}{{\ensuremath{\mathbf m}\xspace}}^T$.
Using this, each exponent in may then be rewritten,

$$\begin{aligned}
 \label{eq:approx}
  -b {{\ensuremath{\mathbf u}\xspace}}_i^T D {{\ensuremath{\mathbf u}\xspace}}_i
  &\approx& -b\lambda_1{{\ensuremath{\mathbf u}\xspace}}_i^T\left({{\ensuremath{\mathbf m}\xspace}}{{\ensuremath{\mathbf m}\xspace}}^T\right){{\ensuremath{\mathbf u}\xspace}}_i \\
  &=& -b \lambda_1 \left({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}\right)^2 \\
  &=& -k \left({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}\right)^2 ,\end{aligned}$$

where the scalar $k$ concentration parameter determines the degree of
anisotropy. Finally, the general model may be restated:

$$\label{eq:watson_model}
  s_i = A \sum_j w_j e^{-k_j ({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}_j)^2} ,$$

where $A$ is a normalization constant such that
${\ensuremath{\|{{\ensuremath{\mathbf s}\xspace}}\|}}=1$. For purposes
of comparison, this normalization will also be done to signals obtained
from scanner. Note that, while the diffusion tensor requires six
parameters, these Watson functions require four parameters: three for
the orientation vector ${{\ensuremath{\mathbf m}\xspace}}$ and one
concentration parameter $k$. Employing a spherical representation can
further reduce the unit vector <span><span>$\mathbf m$</span></span>to
two spherical coordinates. demonstrates how adjusting the $k$-value
produces different diffusion patterns, and illustrates two multi-fiber
configurations.

[fig:watson]

From this general mixture model, we choose to start with a restricted
form involving two equally-weighted Watson functions. This choice is
guided by several previous studies. Behrens <span>et
al</span>@Behrens2007 showed that at a $b$-value of 1000 ms/mm$^2$ the
maximum number of detectable fibers is two. Several other studies have
also found two-fiber models to be sufficient @Tuch2002
[@Kreher2005; @Zhan2006; @Peled2006]. Using this as a practical
guideline, we started with a mixture of two Watson functions as our
local fiber model. Further, following the study of @Zhan2006, we assume
an equal combination (50%-50%) of the two Watson functions. While the
effect of this second choice appears to have little to no effect on
experiments, we have yet to quantify any potential loss in accuracy.
These assumptions leave us with the following two-fiber model used in
this study:

$$\label{eq:2W_model}
  s_i = \frac{A}{2} \left(e^{ -k_1 ({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}_1)^2 } + e^{ -k_2 ({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}_2)^2 } \right) .$$

where $k_1$ and ${{\ensuremath{\mathbf m}\xspace}}_1$ parameterize the
first Watson function, $k_2$ and ${{\ensuremath{\mathbf m}\xspace}}_2$
parameterize the second, and $A$ is again a normalization constant such
that ${\ensuremath{\|{{\ensuremath{\mathbf s}\xspace}}\|}}=1$. Thus, the
equally-weighted two-fiber model is fully described by the following
parameters: $k_1$, ${{\ensuremath{\mathbf m}\xspace}}_1$, $k_2$,
${{\ensuremath{\mathbf m}\xspace}}_2$. Extending off the two-Watson
model, we can directly formulate the equally-weighted three-Watson
model:

$$\label{eq:3W_model}
  s_i = \frac{A}{3} \sum_{j=1}^3 e^{-k_j ({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}_j)^2} ,$$

with the additional parameters $k_3$ and
${{\ensuremath{\mathbf m}\xspace}}_3$.

Finally, from such parameters, Rathi <span>et al</span>@Rathi2009mia_w
describe how one may compute the ODF analytically by applying the
Funk-Radon transform directly to . The ODF can be reconstructed directly
from the same parameters describing the signal without a separate
estimation process. For the two-Watson model () the ODF is approximated
by,

$$\label{eq:2W_model_odf}
  f_i = \frac{B}{2} \left(e^{ -\tfrac{k_1}{2} (1-({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}_1)^2) }
                        + e^{ -\tfrac{k_2}{2} (1-({{\ensuremath{\mathbf u}\xspace}}_i^T {{\ensuremath{\mathbf m}\xspace}}_2)^2) } \right) ,$$

where $B$ is a normalization factor such that $\sum_i f_i = 1$.

Estimating the fiber model {#sec:estimation}
--------------------------

Given the measured scanner signal at a particular voxel, we want to
estimate the underlying model parameters that explain this signal. As in
streamline tractography, we treat the fiber as the trajectory of a
particle which we trace out. At each step, we examine the measured
signal at that position, estimate the underlying model parameters, and
propagate forward in the most consistent direction. illustrates this
filtering process.

To use a state-space filter for estimating the model parameters, we need
the application-specific definition of four filter components:

The system state <span><span>$\mathbf x$</span></span>: the model
parameters

The state transition $f[\cdot]$: how the model changes as we trace the
fiber

The observation $h[\cdot]$: how the signal appears given a particular
state

The measurement <span><span>$\mathbf y$</span></span>: the actual signal
obtained from the scanner

For our state, we directly use the model parameters, thus the two-fiber
model in has the following state vector:

$$\label{eq:state}
  {{\ensuremath{\mathbf x}\xspace}}= {\ensuremath{[\, {{\ensuremath{\mathbf m}\xspace}}_1 \;\; k_1 \;\; {{\ensuremath{\mathbf m}\xspace}}_2 \;\; k_2  \,]}\xspace}^T,
  \quad
  {{\ensuremath{\mathbf m}\xspace}}\in {\ensuremath{\mathbb S}}^2, k \in {\ensuremath{\mathbb R}}.$$

While each ${{\ensuremath{\mathbf m}\xspace}}$ could be represented in a
reduced spherical form, the antipodes of the spherical parameterization
would then introduce nonlinearities which complicate estimation. For the
state transition we assume identity dynamics; the local fiber
configuration does not undergo drastic change from one position to the
next. Our observation is the signal reconstruction,
${{\ensuremath{\mathbf y}\xspace}}={{\ensuremath{\mathbf s}\xspace}}={\ensuremath{[\, s_1,...,s_n \,]}\xspace}^T$
using $s_i$ from , and our measurement is the actual signal interpolated
directly from the diffusion weighted images at the current position.

Since the signal reconstruction is a nonlinear process, we employ an
unscented Kalman filter to perform nonlinear estimation. Similar to
classical linear Kalman filtering, the unscented version seeks to
reconcile the predicted state of the system with the measured state and
addresses the fact that those two processes (prediction and measurement)
may be nonlinear or unknown. It does this in two phases: first it uses
the system transition model to predict the next state and observation,
and then it uses the new measurement to correct that state estimate. In
what follows, we present the algorithmic application of the filter. For
more thorough treatments, see @Julier2004 [@Merwe2003].

It is important to note two alternative techniques for nonlinear
estimation. First, particle filters are commonly used to provide a
multi-modal estimate of unknown systems. With respect to an
$n$-dimensional state space, particle filters require the number of
particles to be exponential to properly explore the state space. In
contrast, the unscented filter requires only $2n+1$ particles (sigma
points) for a Gaussian estimate that space. Further, for many slowly
varying systems, the multi-modal estimate is unnecessary: from one voxel
to the next, fibers tend not to change direction drastically. Second, an
extended Kalman filter may also be used to provide a Gaussian estimate
after linearizing the system; however, the unscented Kalman filter
provides a more accurate estimate with equivalent computational cost and
altogether avoids the attempt at linearization @Julier2004
[@Merwe2003; @Lefebvre2004].

Suppose the system of interest is at time $t$ and we have a Gaussian
estimate of its current state with mean,
${{\ensuremath{\mathbf x}\xspace}}_t \in {\ensuremath{\mathbb R}}^n$,
and covariance, $P_t \in
{\ensuremath{\mathbb R}}^{n \times n}$. Prediction begins with the
formation of a set
${{\ensuremath{\mathbf X}\xspace}}_t=\{{{\ensuremath{\mathbf x}\xspace}}_i\} \subset {\ensuremath{\mathbb R}}^n$
of $2n+1$ *sigma point* states with associated convex weights,
$w_i \in {\ensuremath{\mathbb R}}$, each a perturbed version of the
current state. We use the covariance, $P_t$, to distribute this set:

$${{\ensuremath{\mathbf x}\xspace}}_0 = {{\ensuremath{\mathbf x}\xspace}}_t
  \qquad
  w_0 = \kappa/(n+\kappa)
  \qquad
  w_i = w_{i+n} = \tfrac{1}{2(n+\kappa)}$$

$$\label{eq:sigma_points}
  {{\ensuremath{\mathbf x}\xspace}}_{i}   = {{\ensuremath{\mathbf x}\xspace}}_t + \left[\sqrt{(n+\kappa)P_t}\right]_i
  \quad
  {{\ensuremath{\mathbf x}\xspace}}_{i+n} = {{\ensuremath{\mathbf x}\xspace}}_t - \left[\sqrt{(n+\kappa)P_t}\right]_i$$

where $[A]_i$ denotes the $i^\text{th}$ column of matrix $A$ and
$\kappa$ is an adjustable scaling parameter ($\kappa = 0.01$ in all
experiments). Next, this set is propagated through the state transition
function,
$\hat{{{\ensuremath{\mathbf x}\xspace}}}=f[{{\ensuremath{\mathbf x}\xspace}}]
\in {\ensuremath{\mathbb R}}^n$, to obtain a new predicted sigma point
set:
${{\ensuremath{\mathbf X}\xspace}}_{t+1|t}=\{f[{{\ensuremath{\mathbf x}\xspace}}_i]\}=\{\hat{{{\ensuremath{\mathbf x}\xspace}}}_i\}$.
Since in this study we assume the fiber configuration does not change
drastically as we follow it from one voxel to the next, we may write
this identity transition as,
${{\ensuremath{\mathbf x}\xspace}}_{t+1|t} = f[{{\ensuremath{\mathbf x}\xspace}}_t] =
{{\ensuremath{\mathbf x}\xspace}}_t $. These are then used to calculate
the predicted system mean state and covariance,

$$\bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t} = \sum_i w_i ~ \hat{{{\ensuremath{\mathbf x}\xspace}}}_i ,$$

$$\label{eq:Pxx}
  P_{xx} = \sum_i w_i \left(\hat{{{\ensuremath{\mathbf x}\xspace}}}_i - \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t}\right)
                     \left(\hat{{{\ensuremath{\mathbf x}\xspace}}}_i - \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t}\right)^T
           + Q ,$$

where $Q$ is the injected process noise bias used to ensure a non-null
spread of sigma points and a positive-definite covariance. This
procedure comprises the *unscented transform* used to estimate the
behavior of a nonlinear function: spread sigma points based on your
current uncertainty, propagate those using your transform function, and
measure their spread.

To obtain the predicted observation, we again apply the unscented
transform this time using the predicted states,
${{\ensuremath{\mathbf X}\xspace}}_{t+1|t}$, to estimate what we expect
to observe from the hypothetical measurement of each state:
${{\ensuremath{\mathbf y}\xspace}}=h[\hat{{{\ensuremath{\mathbf x}\xspace}}}] \in {\ensuremath{\mathbb R}}^m$.
Keep in mind that, for this study, our observation is the signal
reconstruction from , and the measurement itself is the
diffusion-weighted signal, ${{\ensuremath{\mathbf s}\xspace}}$,
interpolated at the current position. From these, we obtain the
predicted set of observations,
${{\ensuremath{\mathbf Y}\xspace}}_{t+1|t} = \{ h[\hat{{{\ensuremath{\mathbf x}\xspace}}}_i] \} = \{ {{\ensuremath{\mathbf y}\xspace}}_i \}$,
and may calculate its weighted mean and covariance,

$$\bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t} = \sum_i w_i ~ \hat{{{\ensuremath{\mathbf y}\xspace}}}_i ,$$

$$\label{eq:Pyy}
  P_{yy} = \sum_i w_i \left(\hat{{{\ensuremath{\mathbf y}\xspace}}}_i - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t}\right)
                     \left(\hat{{{\ensuremath{\mathbf y}\xspace}}}_i - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t}\right)^T
           + R ,$$

where $R$ is the injected measurement noise bias again used to ensure a
positive-definite covariance. The cross correlation between the
estimated state and measurement may also be calculated:

$$\label{eq:Pxy}
  P_{xy} = \sum_i w_i \left(\hat{{{\ensuremath{\mathbf x}\xspace}}}_i - \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t}\right)
                     \left(\hat{{{\ensuremath{\mathbf y}\xspace}}}_i - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t}\right)^T .$$

[t]

[alg:ukf]

[1] Form weighted sigma points
${{\ensuremath{\mathbf X}\xspace}}_t=\{w_i, {{\ensuremath{\mathbf x}\xspace}}_i\}_{i=0}^{2n}$
around current mean ${{\ensuremath{\mathbf x}\xspace}}_t$ and covariance
$P_t$ with scaling factor $\zeta$

$${{\ensuremath{\mathbf x}\xspace}}_0 = {{\ensuremath{\mathbf x}\xspace}}_t
      \qquad
      {{\ensuremath{\mathbf x}\xspace}}_i    = {{\ensuremath{\mathbf x}\xspace}}_t + [\sqrt{\zeta P_t}]_i
      \qquad
      {{\ensuremath{\mathbf x}\xspace}}_{i+n} = {{\ensuremath{\mathbf x}\xspace}}_t - [\sqrt{\zeta P_t}]_i$$

Predict the new sigma points and observations

$${{\ensuremath{\mathbf X}\xspace}}_{t+1|t} = f[{{\ensuremath{\mathbf X}\xspace}}_t]   \qquad   {{\ensuremath{\mathbf Y}\xspace}}_{t+1|t} = h[{{\ensuremath{\mathbf X}\xspace}}_{t+1|t}]$$

Compute weighted means and covariances, <span>*e.g*</span>

$$\bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t} = \sum_i w_i ~ {{\ensuremath{\mathbf x}\xspace}}_i
      \qquad
      P_{xy} = \sum_i w_i ({{\ensuremath{\mathbf x}\xspace}}_i - \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t})({{\ensuremath{\mathbf y}\xspace}}_i - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t})^T$$

Update estimate using Kalman gain $K$ and scanner measurement
${{\ensuremath{\mathbf y}\xspace}}_t$

$$K = P_{xy}P_{yy}^{-1}
      \qquad
      {{\ensuremath{\mathbf x}\xspace}}_{t+1} = \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t} + K({{\ensuremath{\mathbf y}\xspace}}_t - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t})
      \qquad
      P_{t+1} = P_{xx} - K P_{yy} K^T$$

As is done in the classic linear Kalman filter, the final step is to use
the Kalman gain, $K = P_{xy}P_{yy}^{-1}$, to correct our prediction and
provide us with the final estimated system mean and covariance,

$$\label{eq:x_}
  {{\ensuremath{\mathbf x}\xspace}}_{t+1} = \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t} + K({{\ensuremath{\mathbf y}\xspace}}_t - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t})$$

$$\label{eq:P_}
  P_{t+1} = P_{xx} - K P_{yy} K^T ,$$

where
${{\ensuremath{\mathbf y}\xspace}}_t \in {\ensuremath{\mathbb R}}^m$ is
the actual signal measurement taken at this time. summarizes this
algorithm for unscented Kalman filtering.

The algorithm {#sec:alg}
-------------

To summarize the proposed technique, we are using the unscented Kalman
filter to estimate the local model parameters as we trace out each
fiber. For each fiber, we maintain the position at which we are
currently tracing it and the current estimate of its model parameters
(mean and covariance). At each iteration of the algorithm, we predict
the new state, which in this case is simply identity
(${{\ensuremath{\mathbf x}\xspace}}_{t+1|t}={{\ensuremath{\mathbf x}\xspace}}_t$)
as we assume the fiber does not change drastically in character from one
position to the next. Our actual measurement
${{\ensuremath{\mathbf y}\xspace}}_t$ in is the diffusion-weighted
signal, <span><span>$\mathbf s$</span></span>, recorded by the scanner
at this position. At subvoxel positions we interpolate directly on the
diffusion-weighted images. With these, we step through the equations
above to find the new estimated model parameters,
${{\ensuremath{\mathbf x}\xspace}}_{t+1}$. Last, we use path integration
to move a small step in the most consistent of principal diffusion
directions, and then we repeat these steps from that new location.
outlines the feedback loop between filtering and tractography.

[t]

[alg:loop]

[1] Form the sigma points ${{\ensuremath{\mathbf X}\xspace}}_t$ around
${{\ensuremath{\mathbf x}\xspace}}_t$ Predict the new sigma points
${{\ensuremath{\mathbf X}\xspace}}_{t+1|t}$ and observations
${{\ensuremath{\mathbf Y}\xspace}}_{t+1|t}$ Compute weighted means and
covariances,
<span>*e.g*</span>$\bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t}$,
$P_{xy}$ Update estimate (${{\ensuremath{\mathbf x}\xspace}}_{t+1}$,
$P_{t+1}$) using scanner measurement
(${{\ensuremath{\mathbf y}\xspace}}_t$) Proceed in most consistent
direction ${{\ensuremath{\mathbf m}\xspace}}_j$

Results
=======

The supporting results of this dissertation are divided into three
sections. First, we focus on the equally-weighted diffusion tensor model
treating both two and three component models. Second, we switch to the
Watson directional function as a model and repeat several of these
experiments. Last, we extend this technique to weighted mixtures and
employ a constrained filter. Also, a Python implementation of filtered
tractography is made available in Slicer 3.6.

Tensors
-------

We begin with synthetic experiments to validate our technique against
ground truth. After constructing a set of crossing fiber fields, we
perform tractography and examine the underlying orientations and
branchings (). We then turn to more quantitative experiments over a
broad range of angles and component weightings, and we confirm that our
approach accurately recognizes crossing fibers () and provides a
superior estimate of the diffusion process (). We then demonstrate
estimation of three-fiber crossings (). Lastly, we examine a real
dataset to demonstrate how causal estimation is able to pick up fibers
and branchings known to exist <span>*in vivo*</span>yet absent using an
assortment of other techniques (). This section follows our work found
in @malcolm2010tmi.

[fig:T~s~ynthetic]

Following the experimental method of generating synthetic data found in
@Tournier2004 [@Descoteaux2009tmi; @Schultz2008], we pulled from our
real data set the 300 voxels with highest fractional anisotropy (FA) and
compute the average eigenvalues among these voxels:
$\{1200, 100, 100\}\mu$m$^2$/msec (FA=0.91). We generated synthetic MR
signals according to using these eigenvalues to form an anisotropic
tensor at <span><span>$b\!=\!1000$</span></span>s/mm$^2$ using 81
gradient directions uniformly spread on the hemisphere. We assume
$s_0=1$ and introduce Rician noise (<span>SNR $\approx$ 5 dB</span>).
For extra experiments at <span><span>$b\!=\!3000$</span></span>and
alternative noise levels, one can refer to an earlier conference version
of this work @Malcolm2009ipmi.

In these experiments, we compare against several techniques selected to
represent standard alternative models and fitting procedures. First, we
include direct (least-squares) single-tensor estimation and streamline
tractography @Basser2000. Despite its limited utility in crossing and
branching regions, this technique is widely used in both the clinical
and neuroscience communities and provides a baseline for comparison.
Second, we use the same two-tensor model from but estimate the model
parameters using a Levenberg-Marquardt nonlinear least-squares solver.
This shows the effect of filtered estimation versus a standard
alternative scheme. Such techniques depend largely on the
initialization, and so we employ several variants to remove any
uncertainty in initialization. For the synthetic experiments, we
initialize with the ground truth. For the <span>*in
vivo*</span>experiments, we initialize with the single-tensor estimate
@Peled2006 which we loosely refer to as “independent” and we initialize
with the estimate at the previous position which we refer to as
“causal”. Third, we use spherical harmonics for modeling and fiber-ODF
sharpening for peak detection (order $l=8$, regularization $L=0.006$)
@Tournier2004 [@Descoteaux2009tmi]. This provides a comparison with an
independently estimated, model-free representation. Note that this
technique is very similar to spherical deconvolution. We will often
refer to this method as “sharpened spherical harmonics”.

The unscented Kalman filter conveniently requires few parameters.
Specifically, of importance are the matrices for injecting model noise
$Q$ and injecting measurement noise $R$ (see and ). Fortunately, the
relative magnitude of each can be determined off-line from the data
itself, and typically these are formulated as diagonal matrices (zeros
off the diagonal). The injected model noise governs how much variance is
allowed in the model: higher values allow more variation but, pushed too
far, could lead to inaccurate estimation. We found roughly
$q_{{\ensuremath{\mathbf m}\xspace}}\in [0.0015, 0.0030]$ (roughly
2-4<span>$^\circ$</span>) and $q_\lambda \in [25,100]$ to allow an
appropriate amount of angular and diffusive flexibility among our
synthetic and <span>*in vivo*</span>experiments as well as among various
other patients and across other scanner protocols we have encountered.
The injected measurement noise governs how much variance is expected in
the measurement: higher values mean we expect more variance and hence
trust our measurement less. This value depends on the level of physical
noise present which varies depending on the scanner, protocol, or
pre-processing, and so some experimentation may be necessary. However,
in all our experiments thus far, we have found $r_s \in [0.01, 0.03]$ to
quite robust.

### Synthetic tractography {#sec:tracts}

While the independent optimization techniques can be run on individually
generated voxels, care must be taken in constructing reasonable
scenarios to test the causal filter. For this purpose, we constructed an
actual 2D field through which to navigate. In the middle is one long
fiber pathway where the filter begins estimating a single tensor but
then runs into a field of voxels with two crossed fibers at a fixed
angle. We generated several similar fields, each at a different fixed
angle and component weighting. In we show one such field with a
60<span>$^\circ$</span>crossing. In our experiments, fibers start from
the bottom and propagate upward where they encounter the crossing
region. We found that in regions with only one true fiber present (those
outside the crossing here), the second component consistently aligned
with the first.

[fig:angle~3~T]

In we take a closer look at several points along this single fiber as it
passes through the crossing region. We examine the reconstructed ODFs
produced by the filter as well as those produced by spherical harmonic
modeling at those same positions. As reported in @Descoteaux2009tmi
[@Schultz2008], spherical harmonics at
<span><span>$b\!=\!1000$</span></span>begin to not detect the second
component at around 50<span>$^\circ$</span>-60<span>$^\circ$</span>, but
instead report a single angle as seen in one of the middle samples in .
As reported in @Zhan2006 [@Tournier2007; @Schultz2008], a close
examination of the reported axes shows this bias toward a single
averaged axis. In contrast, the filtered results are consistent and
accurate. One can note a slight deflection upon entry of the crossing
region as the filter attempts to maintain smooth estimates. The
deflection is lessened upon exit since the single component allows for
the most stable model fit.

[fig:T~t~c]

### Angular resolution {#sec:angle}

Having verified the underlying behavior, we then began a more
comprehensive evaluation and quantified the estimated angle within the
crossing regions. Synthetic crossing fields were constructed with a
range of crossing angles and weighting combinations. In each row is a
different weighting: top 50-50, middle 60-40, bottom 70-30. Each graph
then plots the angular error as a function of crossing angle, from
15<span>$^\circ$</span>to 90<span>$^\circ$</span>. Within the crossing
regions, we compared the performance among direct single-tensor
estimation, sharpened spherical harmonics, a nonlinear solver
(Levenberg-Marquardt), and the proposed filter. By varying the size of
the crossing regions in or the number of fibers run, we ensured that
each technique performed estimation on at least 500 voxels to produce
consistent trendlines across this wide range of angles.

shows the estimated separating angle reported using spherical harmonics
*(red)*, the nonlinear solver *(blue)*, and the proposed filter
*(black)*. Over each technique’s series of estimations, the trendlines
indicate the mean error while the bars indicate one standard deviation.
Consistent with the synthetic results reported in @Descoteaux2009tmi
[@Descoteaux2007mrm], spherical harmonics are generally unable to detect
and resolve angles below 50<span>$^\circ$</span>for
<span><span>$b\!=\!1000$</span></span>or below
40<span>$^\circ$</span>for <span><span>$b\!=\!3000$</span></span>. With
a perfect initialization, the Levenberg-Marquardt solver is closer to
the solution but shows significant variance from the noise. In contrast,
the filtered approach statistically estimates the true underlying signal
and so is capable of resolving angles down to the range of
20-30<span>$^\circ$</span>with 5<span>$^\circ$</span>error in the
equally weighted field (50-50) *(top row)* with performance degrading
little in the 60-40 field *(middle)*. In the 70-30 field *(bottom)*, the
trendlines begin to show an asymptotic limit to performance, yet the
filter is the only method capable of reasonable estimates at large
angles. See @Malcolm2009ipmi for additional experiments at both
<span><span>$b\!=\!1000$</span></span>and
<span><span>$b\!=\!3000$</span></span>. From these trendlines, one can
conclude that the filtered approach provides accurate and consistent
angular resolution at crossing angles far below independent estimation.

### Estimated quantities {#sec:quantities}

While angular resolution is important in accurately resolving paths, the
underlying estimated quantities are important in the analysis of these
pathways. We focus on three scenarios: a single fiber (no crossing), a
45<span>$^\circ$</span>crossing, and an orthogonal
90<span>$^\circ$</span>crossing. In each scenario we hold constant the
primary eigenvalue and adjust the minor eigenvalues to produce fields
with a range of diffusion properties. Further, we adjust the crossing
weights as in the earlier experiments. We compare among the three
techniques estimating diffusion quantities: direct single-tensor
estimation *(green)*, the two-tensor model with the nonlinear solver
(Levenberg-Marquardt initialized with ground truth) *(blue)*, and the
two-tensor model with the proposed filter *(black)*. And finally we
judge these techniques using the error in estimated fractional
anisotropy (FA), one of the most common proxy measures for diffusion.

The fields with a single fiber (no crossing) serve as another baseline.
shows that both direct estimation and the filter provide tight estimates
of the true FA. Note the increased variance incurred in the nonlinear
solver due to noise. In , single-tensor estimation begins to show a
bias. Levenberg-Marquardt begins to deteriorate slightly, but the filter
continues to provide accurate and consistent estimates. Lastly, in ,
single-tensor estimation provides erroneous estimates while only the
multi-component techniques are able to maintain accuracy. Note again how
the filter provides tight estimates.

[fig:T~c~c]

### Volume fractions {#sec:weights}

In the current implementation, we have chosen a model without weights
(). To examine this assumption, we included experiments over weighted
fields to see the effect on estimation. Each row of shows a different
weighting combination: equal in the top row to most asymmetric in the
bottom row.

Despite the equally-weighted assumption, shows that the filter is
capable of correctly resolving angles in the range of 60-40 and only the
most orthogonal angles at 70-30. In all of these runs, the filter
maintained tracing of the dominant fiber, drifting little in the most
asymmetric cases. At 80-20 no technique was able to reliably detect the
minor component.

While the filter had trouble picking up the minor component in the more
asymmetric cases, -[fig:lambda~t~90] shows that it maintained accurate
estimates of the diffusion processes, perhaps the most important
consideration. In these regions where the model does not explicitly fit
the data, we found that the filter compensates by adjusting the
eigenvalues. For example, changes in the eigenvalues could be
interpreted as weights: $
 e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D {{\ensuremath{\mathbf u}\xspace}}_i }
   = e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T (D_a + D_b) {{\ensuremath{\mathbf u}\xspace}}_i }
   = e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_a {{\ensuremath{\mathbf u}\xspace}}_i } e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_b {{\ensuremath{\mathbf u}\xspace}}_i }
   = w e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_b {{\ensuremath{\mathbf u}\xspace}}_i } .
$ In each of the unequally weighted crossings, the filter increases the
eigenvalues of the primary component and decreases those of the minor
component in order to fit the signal with much the same effect as
increasing or decreasing weights. Since all techniques appear to
deteriorate beyond the 70-30 weighting, it appears that components of
less than 30% contribution will not be reliably detected despite
incorporating weights.

The unscented Kalman filter itself places no constraint on the state
space except what is statistically likely given the current estimate:
all values in the state vector are free to take any value in
<span>$\mathbb R$</span>. In our current implementation, we restricted
${{\ensuremath{\mathbf m}\xspace}}$ to ${\ensuremath{\mathbb S}}^2$ and
${{\ensuremath{\lambda_1}} \xspace},{{\ensuremath{\lambda_2}} \xspace}> 0$.
However, this is an ad hoc solution and we have experimented with
applying causal filters capable of estimating under model constraints,
<span>*e.g*</span>positive eigenvalues, convex weights @malcolm2009cukf.

### Three-fiber crossings {#sec:3T}

Resolving three-fiber crossings has proven difficult for many
techniques, especially at the lower $b$-values typically used in
clinical scans. For example, @Tuch2002 found the general multi-tensor
model to be unstable for three or more components using data at
<span>$b\!=\!1077$</span> with 126 gradients. @Bergmann2006 only
reported results for up to two-tensors (<span>$b\!=\!700$</span>, 30
gradients). In simulation, @Behrens2007 found that $b$-values at upwards
of 3000-4000 s/mm$^2$ were required for detecting more than two fibers
and none were found <span>*in vivo*</span>(<span>$b\!=\!1000$</span>, 60
gradients). Further, many studies specifically use at most two
orientations @Alexander2001 [@Kreher2005; @Peled2006]. However,
spherical shell techniques have had some success in resolving
three-fiber crossings. @Tournier2004 reported such crossings using
spherical deconvolution (<span>$b\!=\!2971$</span>, 60 gradients). Most
recently, @Schultz2008 demonstrated tensor decomposition as a promising
technique for resolving such configurations (<span>$b\!=\!1000$</span>,
60 gradients).

Following the synthetic experimental setup of @Schultz2008, we
constructed an additional set of synthetic fields this time with three
equally-weighted Gaussian components. Similar to the synthetic fields
shown in , one fiber is angled up and is the intended orientation to
track through the region, but here the other two orientations were set
so that the endpoints of the three principal axes formed an equilateral
triangle with any two separated by the specified angle. With this setup,
shows that the filtered approach provides an accurate estimate that
reaches to roughly 45<span>$^\circ$</span>compared to
60-65<span>$^\circ$</span>for spherical harmonics. A significant bias is
apparent at more acute angles using either technique.

![Successive snapshots of filtered two-tensor tracing from both the
corpus callosum *(red)* and internal capsule *(yellow)*, viewed from
right hemisphere. Here we see the lateral pathways from the corpus cross
the motor tracts from the internal capsule.](000018 "fig:") ![Successive
snapshots of filtered two-tensor tracing from both the corpus callosum
*(red)* and internal capsule *(yellow)*, viewed from right hemisphere.
Here we see the lateral pathways from the corpus cross the motor tracts
from the internal capsule.](000021 "fig:") ![Successive snapshots of
filtered two-tensor tracing from both the corpus callosum *(red)* and
internal capsule *(yellow)*, viewed from right hemisphere. Here we see
the lateral pathways from the corpus cross the motor tracts from the
internal capsule.](000024 "fig:") ![Successive snapshots of filtered
two-tensor tracing from both the corpus callosum *(red)* and internal
capsule *(yellow)*, viewed from right hemisphere. Here we see the
lateral pathways from the corpus cross the motor tracts from the
internal capsule.](000027 "fig:") ![Successive snapshots of filtered
two-tensor tracing from both the corpus callosum *(red)* and internal
capsule *(yellow)*, viewed from right hemisphere. Here we see the
lateral pathways from the corpus cross the motor tracts from the
internal capsule.](000035 "fig:")

[fig:cc~i~c]

![The three-tensor filtered approach is able to trace through this
intersection of the corticospinal tract *(blue)*, corpus callosum
*(red)*, and superior longitudinal fasciculus
*(green)*.](3cross_context "fig:") ![The three-tensor filtered approach
is able to trace through this intersection of the corticospinal tract
*(blue)*, corpus callosum *(red)*, and superior longitudinal fasciculus
*(green)*.](3cross "fig:")

[fig:intersection~3~T]

### <span>*In vivo*</span>tractography {#sec:2T_real}

We next test our approach on a real human brain scan acquired on a
3-Tesla GE system using an echo planar imaging (EPI) diffusion weighted
image sequence. A double echo option was used to reduce eddy-current
related distortions. To reduce impact of EPI spatial distortion, an
eight channel coil was used to perform parallel imaging using Array
Spatial Sensitivity Encoding Techniques (GE) with a SENSE-factor
(speed-up) of 2. Acquisitions have 51 gradient directions with
<span>$b\!=\!900$</span> and eight baseline scans with
<span>$b\!=\!0$</span>. The original GE sequence was modified to
increase spatial resolution, and to further minimize image artifacts.
The following scan parameters were used: TR 17000 ms, TE 78 ms, FOV 24
cm, 144x144 encoding steps, 1.7 mm slice thickness. All scans had 85
axial slices parallel to the AC-PC line covering the whole brain. In
addition, <span>$b\!=\!0$</span> field inhomogeneity maps were collected
and calculated.

We first focus on fibers originating in the corpus callosum.
Specifically, we seek to trace out the lateral transcallosal fibers that
run through the corpus callosum out to the lateral gyri. It is known
that single-tensor streamline tractography only traces out the dominant
pathways forming the U-shaped callosal radiation (, [fig:T~1~T~t~c], and
[fig:T~1~T~c~c]). Several studies document this phenomena, among them
the works of Descoteaux, Schultz, <span>et al</span>@Descoteaux2009tmi
[@Schultz2008] have side-by-side comparisons. These fibers have been
reported in using diffusion spectrum imaging @Hagmann2004, probabilistic
tractography @Kaden2007 [@Anwander2007; @Descoteaux2009tmi], and more
recently with tensor decomposition @Schultz2008.

We start with two basic experiments: first examining the tracts
surrounding a single coronal slice and second looking at all tracts
passing through the corpus callosum. We seed each algorithm multiple
times in voxels at the intersection of the mid-sagital plane and the
corpus callosum. We terminate fibers when the generalized fractional
anisotropy of the estimated signal (std/rms) fell below 0.1. To explore
branchings found using the proposed technique, we consider a component
to be branching if it was separated from the primary component by less
than 40<span>$^\circ$</span>with $FA\ge0.15$. Similarly, with sharpened
spherical harmonics, we consider it a branch if we find additional
maxima over the same range. Unless otherwise stated, in all experiments
we follow the primary fiber from the seeding and its branches,
<span>*i.e*</span>one level of branching. We do not go on to follow the
branches on those secondary fibers. While such heuristics are somewhat
arbitrary, we found little qualitative difference in adjusting these
thresholds.

To demonstrate the flexibility of the proposed filtering strategy with
respect to model choice, we use both the two-tensor fiber model () and
the three-tensor fiber model (). While this introduced differences in
the quantity of branchings detected, we found that using either model
resulted in generally the same pathways. This suggests that the
filtering itself accounted for most of the differences compared to the
other techniques, more so than the choice of two or three components in
the fiber model. Further, it is important to note that despite the more
complicated multi-fiber models, the filter provides stable and
consistent estimates of the appropriate number of compartments. For
example, when tracing using the two-tensor fiber through a one-tensor
region, the filter overlaps both tensors to fit the signal (above and
below the crossing region in ).

For the first experiment, shows tracts originating from within a few
voxels intersecting a particular coronal slice. As a reference, we use a
coronal slice showing the intensity of fractional anisotropy (FA) placed
a few voxels behind the seeded coronal position. Keeping in mind that
these fibers are intersecting or are in front of the image plane, this
roughly shows how the fibers navigate the areas of high anisotropy
(bright regions). Comparable to the results in @Descoteaux2009tmi
[@Schultz2008], shows that sharpened spherical harmonics only pick up a
few fibers intersecting the U-shaped callosal radiata. In contrast, our
proposed method traces out many pathways consistent with the apparent
anatomy using either the two-fiber or three-fiber model. To emphasize
transcallosal tracts, we color as blue those fibers exiting a corridor
of $\pm22\,\text{mm}$ around the mid-sagittal plane. To explore the
maximum connectivity found using each approach, this is the one
experiment where we followed secondary branches,
<span>*i.e*</span>primary fibers, their branches, and finally branches
off those fibers. shows a view of the whole brain to see the overall
difference between the different methods.

[fig:slf]

[fig:cing]

[fig:ioff]

[fig:ioff~t~op]

We next combined the filtered tracings from the corpus callosum with
those from the internal capsule to demonstrate practically how the
filter is able to push through areas of crossing. shows successive
snapshots as two-tensor filtered tracings from the corpus callosum
*(red)* infiltrate those originating in the internal capsule *(yellow)*.
Since other methods were unable to reconstruct such dense penetration,
it is our hope that this method of multi-tensor tractography will
provide rich information in connectivity studies. takes a closer look at
one area where these fiber pathways merge and shows fragments from the
fibers traversing through this region using the three-tensor model.

In the experiments thus far, the decision to branch is somewhat ad hoc.
An alternative approach to avoid such thresholds is to perform
full-brain tractography following only one path and then select fibers
using masks. Along these lines we seeded every voxel with $FA\ge0.15$
using each technique discussed so far. In addition, we included filtered
single-tensor tractography to contrast to the baseline single-tensor
streamline.

We select out three well-studied pathways. shows the first pathway, a
portion of the superior longitudinal fasciculus as it crosses the
lateral pathways exposed in earlier figures. Only two techniques were
able to produce this portion of the tract: the causal variant of
Levenberg-Marquardt (initializes each step with previous solution) and
filtered two-tensor tractography. Both share the same general shape, but
the filtered version appears to show a superior reconstruction,
especially where the endpoints insert. Spherical harmonics were unable
to traverse through the lateral crossings here likely due to ambiguous
peaks along this corridor, and the single-tensor models are inherently
incapable of modeling the crossing.

shows the second pathway, the cingulum bundle as it passes through
several gates *(yellow)*. Each technique incurs false-positive fibers at
the ends of the corpus–the genu (anterior) and splenium
(posterior)–where partial voluming often leads to fibers straying onto
the corpus callosum. While all techniques have difficulty making the
posterior bend, they begin to differ in their reconstructions of the
main body. Going from streamline single-tensor to filtered
single-tensor, we see a fuller reconstruction. Levenberg-Marquardt shows
many false positives as it easily finds incorrect minima. Sharpened
spherical harmonics provide an accurate although sparse reconstruction.
Filtered two-tensor provides a full reconstruction although it produces
many false positives.

Last, shows tracing of the interior occipital-fronto fasciculus as it
spreads and inserts into the occipital and temporal lobes. Streamline
single tractography reconstructs the central pathway. Only the filtered
techniques provide consistently deeper penetration into the gray matter
while retaining coherent paths. Further, only the filtered version of
the two-tensor model reconstructs the known minor insertions into the
temporal lobe. This is even more pronounced in the views from above in .

Watson directional functions
----------------------------

We now treat the evaluation of the Watson directional function
separately. Since this model is an approximation of the tensor diffusion
model, we begin with experiments examining reconstruction error as well
as those measuring angular error before moving quickly to <span>*in
vivo*</span>experiments. This section follows our work in
@malcolm2010mia.

[fig:mse~2~W]

Throughout these experiments, we draw comparison to three alternative
techniques. First, we use the same two-Watson model from with a variant
of matching pursuit for brute force, dictionary-based optimization
@Mallat1993. In our implementation, we construct a finite dictionary of
two-Watson signals at a range of various $k$-values, essentially
discretizing the search space across orientations and $k$-values. Given
a new measured signal, the signal from the dictionary with highest inner
product provides an estimate of orientation and concentration. While our
signal is produced at 81 gradient directions, we use 341 directions to
construct the dictionary, thus any error is due to the method’s
sensitivity to noise and discretization. Note that by using 341
orientation directions there is roughly an 8<span>$^\circ$</span>angular
difference between offset orientations, hence we see that the angular
error is often at most 8<span>$^\circ$</span>. This approach highlights
the effect of using the same model but changing the optimization
technique to one that treats each voxel independently. Second, as in the
tensor experiments above, we use spherical harmonics for modeling
@Tournier2004 and fiber-ODF sharpening for peak detection as described
in @Descoteaux2009tmi (order $l=8$, regularization $L=0.006$). This
provides a comparison with an independently estimated, model-free
representation. Last, when performing tractography on real data, we
again include single-tensor streamline tractography as a baseline.

[fig:2W~a~ngle]

For the Watson model, we found that values on the order of
$q_{{\ensuremath{\mathbf m}\xspace}}= 0.001$ (roughly
2<span>$^\circ$</span>), $q_\lambda = 10$, and $r_s=0.02$ were quite
robust for the appropriate diagonal entries of $Q$ and $R$ (see and ).
Off-diagonal entries were left at zero.

[fig:W~t~c]

[fig:W~t~c~z~oom]

[fig:frontal]

[fig:ic~f~ront]

[fig:ic~t~op]

[fig:ic~s~ide]

### Signal reconstruction and angular resolution {#sec:mse_angle}

In the first experiment, we look at signal reconstruction error. We
calculate the mean squared error of the reconstructed signal,
<span><span>$\mathbf s$</span></span>, against the ground truth signal,
$\hat{{{\ensuremath{\mathbf s}\xspace}}}$ (pure, no noise):
$ {\ensuremath{\|{{\ensuremath{\mathbf s}\xspace}}- \hat{{{\ensuremath{\mathbf s}\xspace}}}\|}}^2 /
{\ensuremath{\|\hat{{{\ensuremath{\mathbf s}\xspace}}}\|}}^2$. In
essence, this is exactly what the filter is trying to minimize: the
error between the reconstructed signal and the measured signal. shows
the results of using the proposed filter, matching pursuit, and
spherical harmonics. Over each technique’s series of estimations, the
trendlines indicate the mean error while the bars indicate one standard
deviation. Spherical harmonics *(red)* appear to produce a smooth fit to
the given noisy data, while matching pursuit *(blue)* shows the effect
of discretization and sensitivity to noise. The two raised areas are a
result of the dictionary being constructed with an
8<span>$^\circ$</span>minimum separation between any pair of
orientations. This experiment demonstrates that the proposed filter
*(black)* accurately and reliably estimates the true underlying signal.

In the second experiment, we looked at the error in angular resolution
by comparing the filtered approach to matching pursuit and sharpened
spherical harmonics. and show the sensitivity of matching pursuit.
Consistent with the results reported in @Descoteaux2009tmi
[@Descoteaux2007mrm], spherical harmonics are generally unable to detect
and resolve angles below 50<span>$^\circ$</span>for
<span><span>$b\!=\!1000$</span></span>or below
40<span>$^\circ$</span>for <span><span>$b\!=\!3000$</span></span>. and
confirm this, respectively. This experiment demonstrates that for
<span><span>$b\!=\!1000$</span></span>, the filtered approach
consistently resolves angles down to 20-30<span>$^\circ$</span>with
5<span>$^\circ$</span>error compared to independent optimization which
fails to reliably resolve below 60<span>$^\circ$</span>with as much as
15<span>$^\circ$</span>error. For
<span><span>$b\!=\!3000$</span></span>, the filtered approach
consistently resolves down to 20-30<span>$^\circ$</span>with
2-3<span>$^\circ$</span>error compared to independent optimization which
cannot resolve below 50<span>$^\circ$</span>with
5<span>$^\circ$</span>error.

### *In vivo* tractography {#sec:W_real}

As in , we first focused on fibers originating in the corpus callosum.
We proceed with two basic experiments: first examining the tracts
surrounding a single coronal slice and second looking at all tracts
passing through the corpus callosum. We seed each algorithm multiple
times in voxels at the intersection of the mid-sagital plane and the
corpus callosum. To explore branchings found using the proposed
technique, we considered a component to be branching if it was separated
from the primary component by less than 40<span>$^\circ$</span>with
$k\ge0.6$. Similarly, with sharpened spherical harmonics, we considered
it a branch if we found additional maxima over the same range. We
terminated fibers when the general fractional anisotropy of the
estimated signal (std/rms) fell below 0.1. While such heuristics are
somewhat arbitrary, we found little qualitative difference in adjusting
these values.

To demonstrate the flexibility of the proposed filtering strategy with
respect to model choice, we use both the two-Watson fiber model () and
the three-Watson fiber model (). While this introduced differences in
the quantity of branchings detected, we found that using either model
resulted in generally finding the same pathways.

For the first experiment, shows tracts originating from within a few
voxels intersecting a particular coronal slice. For a reference
backdrop, we use a coronal slice showing the intensity of fractional
anisotropy (FA) placed a few voxels behind the seeded coronal position.
Keeping in mind that these fibers are intersecting or are in front of
the image plane, this roughly shows how the fibers navigate the areas of
high anisotropy (bright regions). Our proposed method traces out many
pathways consistent with the apparent anatomy using either the two-fiber
or three-fiber model. To emphasize transcallosal tracts, we color as
blue those fibers exiting a corridor of $\pm22\,\text{mm}$ around the
mid-sagittal plane. provides a closer inspection of and where, to
emphasize the underlying anatomy influencing the fibers, we use as a
backdrop the actual coronal slice passing through the voxels used to
seed this run. Such results are obtained in minutes in either our MATLAB
or Python implementations. At each step, the cost of reconstructing the
signal for few sigma points approaches the cost of a few iterations of
weighted least-squares estimation of a single tensor.

For the second experiment, shows a view of the whole brain to see the
overall difference between the different methods. Here again we
emphasize with blue the transcallosal fibers found using the proposed
filter. Comparing and we see several regions that appear to have
different fiber density using the two models. This suggests that
incorporating model selection into filtered approaches may have a
significant effect. To show the various pathways infiltrating the gyri,
provides a closeup of the frontal lobe from above (without blue
emphasis).

Next we examined fibers passing through the internal capsule to trace
out the pathways reaching up into the primary motor cortex at the top of
the brain as well as down into the hippocampal regions near the brain
stem. shows frontal views for each technique with seeding near the
cerebral peduncles *(blue)*. shows this same result from a side view
where we can see that the filtered approach picks up the corticospinal
pathways. Notice that the two-Watson model picks up the temporopontine
and parietopontine tracts and the three-Watson model further reveals the
occipitopontine pathways, another indication that the chosen fiber model
often affects the results. As reported elsewhere @Behrens2007
[@Qazi2008ni], single-tensor tractography follows the dominant
corticospinal tract to the primary motor cortex. The same pathways were
also found with spherical harmonics. shows a view from above where we
use a transverse FA image slice near the top of the brain as a backdrop
so we can focus on the fiber endpoints. From this we can see how each
method infiltrates the sulci grooves, and specifically we see that the
filtered method is able to infiltrate sulci more lateral compared to
single-tensor tractography.

Note that in the region of intersection between the transcallosal
fibers, the corticospinal, and the superior longitudinal fasciculus, the
partial voluming of each of these pathways leads the filter to report
several end-to-end connections that are not necessarily present, e.g.
fibers originating in the left internal capsule do not pass through this
region, through the corpus callosum and then insert into the right motor
cortex. Many of the lateral extensions are callosal fibers that are
picked up while passing through this juncture. It is our hope that such
connections may be avoided with the introduction of weighted mixtures,
alternative filter formulations, or different heuristic choices in the
algorithm.

[fig:W~c~c]

Weighted Mixtures {#ch:weighted}
=================

Summary
-------

Until now we have made use of two-component and three-component models,
all with fixed volume fractions in the general mixture equation (,).
This assumption is forced upon us because update equations of the
standard Kalman filter are unable to enforce the convex relationship
between mixture coefficients. We began in to explore the impact of this
assumption in the presence of mixed fiber populations. A similar problem
arises with the eigenvalues needing to be positive; however, for this a
practical solution was to clamp these values at small positive values
after each update. Our initial efforts at using the log domain to ensure
positivity showed promise as an indirect method of influencing the
positivity of eigenvalues, but still did not address the convexity
constraint @rathi2010mrm.

To treat the problem more directly, we began exploring constrained
filtering where the state estimate is constrained to a subspace of
allowable solutions @Simon2006. In our situation, this ensures that
tensor eigenvalues are positive and the mixture weights are non-negative
and convex.

defines the weighted two-fiber model employed in this section, and
describes how this model fits into the unscented Kalman filter and more
importantly how the constraints are enforced. This chapter follows our
work in @malcolm2009cukf [@Malcolm2009cukf_ext].

### The weighted model {#sec:2TW}

Following the equally-weighted two-tensor model from earlier (), we use
it here in its more general form:

$$\label{eq:2TW_model}
  s_i = s_0 w_1 e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_1 {{\ensuremath{\mathbf u}\xspace}}_i } + s_0 w_2 e^{ -b {{\ensuremath{\mathbf u}\xspace}}_i^T D_2 {{\ensuremath{\mathbf u}\xspace}}_i } ,$$

where $w_1,w_2$ are convex weights and $D_1,D_2$ are each expressible as
$ D = {{\ensuremath{\lambda_1}} \xspace}{{\ensuremath{\mathbf m}\xspace}}{{\ensuremath{\mathbf m}\xspace}}^T + {{\ensuremath{\lambda_2}} \xspace}\left({\ensuremath{\mathbf p}\xspace}{\ensuremath{\mathbf p}\xspace}^T + {\ensuremath{\mathbf q}\xspace}{\ensuremath{\mathbf q}\xspace}^T\right), $
with
${{\ensuremath{\mathbf m}\xspace}},{\ensuremath{\mathbf p}\xspace},{\ensuremath{\mathbf q}\xspace} \in {\ensuremath{\mathbb S}}^2$
forming an orthonormal basis aligned to the principal diffusion
direction <span><span>$\mathbf m$</span></span>. The free model
parameters are then ${{\ensuremath{\mathbf m}\xspace}}_1$,
${{\ensuremath{\lambda_1}} \xspace}_1$,
${{\ensuremath{\lambda_2}} \xspace}_1$, $w_1$,
${{\ensuremath{\mathbf m}\xspace}}_2$,
${{\ensuremath{\lambda_1}} \xspace}_2$,
${{\ensuremath{\lambda_2}} \xspace}_2$, and $w_2$. Lastly, we wish to
constrain this model to have positive eigenvalues and convex weights
($w_1,w_2\ge0$ and $w_1+w_2=1$).

### Constrained estimation {#sec:cukf}

As in the standard estimation process, we begin with the
application-specific definition of four filter components. The only
difference here is for that we directly include the weighting parameters
for the two-tensor model in :

$$\label{eq:2TW_state}
  {{\ensuremath{\mathbf x}\xspace}}= {\ensuremath{[\,  {{\ensuremath{\mathbf m}\xspace}}_1 \;\; {{\ensuremath{\lambda_1}} \xspace}_1 \;\; {{\ensuremath{\lambda_2}} \xspace}_1 \;\; w_1 \;\;
            {{\ensuremath{\mathbf m}\xspace}}_2 \;\; {{\ensuremath{\lambda_1}} \xspace}_2 \;\; {{\ensuremath{\lambda_2}} \xspace}_2 \;\; w_2  \,]}\xspace}^T ,
  \quad
  {{\ensuremath{\mathbf m}\xspace}}\in{\ensuremath{\mathbb S}}^2, \lambda\in{\ensuremath{\mathbb R}}^{+}, w\in[0,1] .$$

For the state transition we again assume identity dynamics, and our
observation is the signal reconstruction,
${{\ensuremath{\mathbf y}\xspace}}=h[{{\ensuremath{\mathbf x}\xspace}}]={{\ensuremath{\mathbf s}\xspace}}={\ensuremath{[\, s_1,...,s_m \,]}\xspace}^T$
using $s_i$ described by the model in , and our measurement is the
actual signal interpolated directly on the diffusion weighted images at
the current position.

[t]

[alg:cukf]

[1] Form weighted sigma points
${{\ensuremath{\mathbf X}\xspace}}_t=\{w_i, {{\ensuremath{\mathbf x}\xspace}}_i\}_{i=0}^{2n}$
around current mean ${{\ensuremath{\mathbf x}\xspace}}_t$ and covariance
$P_t$ with scaling factor $\zeta$

$${{\ensuremath{\mathbf x}\xspace}}_0 = {{\ensuremath{\mathbf x}\xspace}}_t
      \qquad
      {{\ensuremath{\mathbf x}\xspace}}_i    = {{\ensuremath{\mathbf x}\xspace}}_t + [\sqrt{\zeta P_t}]_i
      \qquad
      {{\ensuremath{\mathbf x}\xspace}}_{i+n} = {{\ensuremath{\mathbf x}\xspace}}_t - [\sqrt{\zeta P_t}]_i$$

Project onto the constrained subspace () Predict the new sigma points
and observations

$${{\ensuremath{\mathbf X}\xspace}}_{t+1|t} = f[{{\ensuremath{\mathbf X}\xspace}}_t]   \qquad   {{\ensuremath{\mathbf Y}\xspace}}_{t+1|t} = h[{{\ensuremath{\mathbf X}\xspace}}_{t+1|t}]$$

Project onto the constrained subspace () Compute weighted means and
covariances, <span>*e.g*</span>

$$\bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t} = \sum_i w_i ~ {{\ensuremath{\mathbf x}\xspace}}_i
      \qquad
      P_{xy} = \sum_i w_i ({{\ensuremath{\mathbf x}\xspace}}_i - \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t})({{\ensuremath{\mathbf y}\xspace}}_i - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t})^T$$

Update estimate using Kalman gain $K$ and scanner measurement
${{\ensuremath{\mathbf y}\xspace}}_t$ Project onto the constrained
subspace ()

$${{\ensuremath{\mathbf x}\xspace}}_{t+1} = \bar{{{\ensuremath{\mathbf x}\xspace}}}_{t+1|t} + K({{\ensuremath{\mathbf y}\xspace}}_t - \bar{{{\ensuremath{\mathbf y}\xspace}}}_{t+1|t})
      \qquad
      P_{t+1} = P_{xx} - K P_{yy} K^T
      \qquad
      K = P_{xy}P_{yy}^{-1}$$

In the standard formulation (), we ignored the constraints on our model.
This results in instabilities: the diffusion tensors may become
degenerate with zero or negative eigenvalues, or the weights may become
negative. To enforce appropriate constraints, one can directly project
any unconstrained state ${{\ensuremath{\mathbf x}\xspace}}$ onto the
constrained subspace @Simon2006. In other words, we wish to find the
state $\hat{{{\ensuremath{\mathbf x}\xspace}}}$ closest to the
unconstrained state ${{\ensuremath{\mathbf x}\xspace}}$ which still
obeys the constraints,
$A\hat{{{\ensuremath{\mathbf x}\xspace}}} \le {\ensuremath{\mathbf b}\xspace}$.
Using $P_t$ as a weighting matrix, this becomes a quadratic programming
problem:

$$\label{eq:constrained}
  \min_{\hat{{{\ensuremath{\mathbf x}\xspace}}}} \ ({{\ensuremath{\mathbf x}\xspace}}- \hat{{{\ensuremath{\mathbf x}\xspace}}})^T P_t^{-1} ({{\ensuremath{\mathbf x}\xspace}}- \hat{{{\ensuremath{\mathbf x}\xspace}}})
  \quad
  \text{subject to}
  \quad
  A\hat{{{\ensuremath{\mathbf x}\xspace}}} \le {\ensuremath{\mathbf b}\xspace} .$$

This projection procedure is applied within unscented Kalman filter
procedure to correct at every place where we move in the state-space:
after spreading the sigma points ${{\ensuremath{\mathbf X}\xspace}}_t$,
after transforming the sigma points
${{\ensuremath{\mathbf X}\xspace}}_{t+1|t}$, and after the final
estimate ${{\ensuremath{\mathbf x}\xspace}}_{t+1}$. See .

In this study, for voxels that can be modeled with only one tensor, we
found it preferable to have both the tensor components similarly
oriented. Upon encountering a region of dispersion, the second component
is poised and ready to begin branching instead of having zero weight and
arbitrary orientation. To favor such solutions, we require the weights
of each of the components to be not just non-negative but also greater
than 0.2, and so, in our current implementation, $A$ and b are
constructed to encode the following state constraints:

$$\label{eq:constraints}
  {{\ensuremath{\lambda_1}} \xspace}_1, {{\ensuremath{\lambda_2}} \xspace}_1, {{\ensuremath{\lambda_1}} \xspace}_2, {{\ensuremath{\lambda_2}} \xspace}_2 > 0
  \qquad
  w_1, w_2 \ge 0.2
  \qquad
  w_1 + w_2 = 1 .$$

Experiments
-----------

We first use experiments with synthetic data to validate our technique
against ground truth. We confirm that our approach accurately recognizes
crossing fibers over a broad range of angles and consistently estimates
the partial volumes (). We then examine a real dataset to demonstrate
how causal estimation is able to pick up fibers and branchings known to
exist *in vivo* yet absent using other techniques ().

In these experiments, we compare against two alternative techniques.
First, we use sharpened spherical harmonics with peak detection as
described in @Descoteaux2007 (order $l=8$, regularization $L=0.006$).
This provides a comparison with an independently estimated nonparametric
representation. Second, when performing tractography on real data, we
also compare against single-tensor streamline tractography for a
baseline.

### Synthetic validation {#sec:w_synthetic}

Following the experimental method of generating multi-compartment
synthetic data found in @Tuch2002 [@Descoteaux2007; @Schultz2008], we
averaged the eigenvalues of the 300 voxels with highest fractional
anisotropy (FA) in our real data set: $\{1200, 100, 100\}\mu$m$^2$/msec.
We used these eigenvalues to generate synthetic MR signals according to
at <span><span>$b\!=\!1000$</span></span>with 81 gradient directions on
the hemisphere and introduced Rician noise (<span>SNR $\approx$ 5
dB</span>).

[fig:w~s~ynthetic]

As before, we constructed a set of two-dimensional fields through which
to navigate (). In the middle is one long pathway where the filter
starts at one end estimating a single tensor but then runs into voxels
with two crossed fibers at a fixed angle and weighting. In this crossing
region we calculated error statistics to compare against sharpened
spherical harmonics. From these synthetic sets, we examined detection
rate, angular resolution, and estimated volume fractions and we plot the
results in . Each column looks at a different primary-secondary
weighting combination, and each row looks at a different metric. In the
top row, we count how many times each technique distinguishes two
separate fibers. The filtered approach *(black)* is able to detect two
distinct fibers at crossing angles far below that using spherical
harmonics *(red)*. Further, the filtered approach maintains such
relatively high detection rates even at 80/20 partial voluming *(far
right column)*. In the middle row, we look at where each technique
reported two fibers and we record the error in estimated angles. From
this, we see that spherical harmonics result in an angular error of
roughly 15<span>$^\circ$</span>at best and fails to detect a second
component at angles below 60<span>$^\circ$</span>. In contrast, the
filtered approach has an error between 5-10<span>$^\circ$</span>and is
able to accurately estimate down to crossing angles of
30<span>$^\circ$</span>. In the bottom row, we look at the primary fiber
weight estimated by the filter. As expected, this estimate is most
accurate closer to 90<span>$^\circ$</span>*(blue line indicates true
weight)*.

### *In vivo* tractography {#sec:w_real}

[fig:TW~t~c]

[fig:TW~c~c]

Our work focuses on fibers originating in the corpus callosum.
Specifically, we sought to trace out the lateral transcallosal fibers
that run through the corpus callosum out to the lateral gyri. It is
known that single-tensor streamline tractography only traces out the
dominant pathways forming the U-shaped callosal radiation while
spherical harmonics only capture some of these pathways @Descoteaux2007
[@Schultz2008]. These fibers have been reported in using diffusion
spectrum imaging @Hagmann2004, probabilistic tractography @Kaden2007
[@Anwander2007; @Descoteaux2007], and more recently with tensor
decomposition @Schultz2008; however, all of these techniques require
lengthy scans at high *b*-values along with significantly more
processing time than streamline techniques.

We begin by seeding each algorithm up to thirty times in voxels at the
intersection of the mid-sagital plane and the corpus callosum. To
explore branchings found using the proposed technique, we considered a
component to be branching if it was separated from the primary component
by less than 40<span>$^\circ$</span>with FA$\ge$0.15 and weight above
0.3. Similarly, with sharpened spherical harmonics, we considered it a
branch if we found additional maxima over the same range. We terminated
fibers when either the generalized fractional anisotropy @Tuch2002 of
the estimated signal fell below 0.1 or the primary component FA fell
below 0.15 or weight below 0.3.

We tested our approach on a human brain scan using a 3-Tesla magnet to
collect 51 diffusion weighted images on the hemisphere at
$b=900\,\text{s/mm}^2$, a scan sequence comparable those of
@Descoteaux2007 [@Schultz2008]. shows tracts originating from within a
few voxels intersecting a chosen coronal slice. Confirming the results
in @Descoteaux2007 [@Schultz2008], sharpened spherical harmonics only
pick up a few fibers intersecting the U-shaped callosal radiata. In
contrast, our proposed algorithm traces out many pathways consistent
with the apparent anatomy. shows a view of the whole corpus callosum
from above. The filtered approach is able to pick up many transcallosal
fibers throughout the corpus callosum as well as infiltrating the
frontal gyri to a greater degree than either alternate technique. To
emphasize transcallosal tracts, we color as blue those fibers exiting a
corridor of $\pm
22\,\text{mm}$ around the mid-sagittal plane.

Summary
-------

In this section, we demonstrated that the constrained approach gives
significantly lower angular error (5-10<span>$^\circ$</span>) in regions
with fiber crossings than using sharpened spherical harmonics
(15-20<span>$^\circ$</span>), and it reliably estimates the partial
volume fractions.

While these initial results are promising, there are several aspects to
be improved. First, without a prior, the weights tend toward the middle
of their range, each tending to hover around 0.6 (halfway between 0.2
and 1.0). The effect is that both components show some presence, even in
strongly coherent fields. The solution might be to induce priors over
their distribution, hopefully forcing them to zero without signal
evidence of a second component. Second, solving the quadratic constraint
is expensive. A practical alternative might be to simply project as
close to the subspace as possible @Simon2006; although, this allows
solutions not necessarily on the subspace and may again open the
possibility for non-convex weights and negative eigenvalues. In
practice, this may still be sufficient.

Validation
==========

Summary
-------

In attempting to reconstruct neural pathways virtually, it is important
to keep in mind the inherent uncertainty in such reconstructions. The
resolution of dMRI scanners is at the level of 3-10mm$^3$ while physical
fiber axons are often an order of magnitude smaller in diameter–a
relationship which leaves much room for error. Some noise or a complex
fiber configuration could simply look like a diffuse signal and cause
probabilistic tractography to stop in its tracks, while a few inaccurate
voxel estimations could easily send the deterministic tracing off course
to produce a false-positive connection. Even global methods could
produce a tract that fits the signal quite well but incidentally jumps
over an actual boundary in one or two voxels it thinks are noise.
Consequently, a common question is: Are these pathways really present?

With this in mind, an active area of study is validating such results.
Since physical dissection often requires weeks of tedious effort, many
techniques have been used for validating these virtual dissections. A
common starting point is to employ synthetic and physical phantoms with
known parameters when evaluating new methods @Poupon2008phantom. When
possible, imaging before and after injecting radio-opaque dyes directly
into the tissue can provide some of the best evidence for comparison
@Lin2003 [@Dauguet2007]. Another powerful approach is to apply bootstrap
sampling or other non-parametric statistical tests to judge the
sensitivity and reproducibility of resulting tractography against
algorithm parameters, image acquisition, and even signal noise
@Lazar2005
[@Jones2005; @Gigandet2008; @Whitcher2008; @Chung2006; @Clayden2007].

To begin validating these filtered approaches, we applied this technique
to a phantom simulating several complex pathway interactions and
highlight tracts passing through several prescribed seed positions. This
section follows our work in @Malcolm2009fc.

Results and Discussion
----------------------

[fig:phantom]

Among the phantoms provided in the 2009 MICCAI Fiber Cup, we present
results using the 3mm version at <span>$b\!=\!1500$</span>
@Poupon2008phantom. Instead of initializing tractography from the
prescribed seed points, we begin by seeding in voxels with nonzero
baseline intensity and terminating tractography when the estimate
becomes isotropic, essentially “full-brain” tractography. From these
potential pathways, shows a representative fiber for each seed point. We
further restricted movement to the image plane.

With the explosion of techniques for mapping connectivity, it is often
difficult to assess the relative merits among various approaches, and
each application has its particular goals. For connectivity studies, one
may only be interested in the final resolved pathway; however, questions
arise such as whether to branch or whether to represent connectivity as
a discrete path or a voxel-to-voxel connectivity matrix possibly telling
more about the relative certainty of connectivity. For tissue studies,
one may be primarily concerned with the estimated microstructure at each
position. Filtered approaches like this provide not only an estimate of
these quantities (mean) but also an additional measure of uncertainty
(variance).

The approach presented here may be considered a local method: at the
current position we estimate a direction and take a step. With such
approaches, one mistake can send the subsequent trajectory off track. We
believe that more global approaches should be considered, ones that take
into account larger portions of the fiber pathway. Further, we believe
that anatomical priors should be incorporated. Such a progression of
techniques may be considered analogous to how level set methods
developed from local edge-based computations, to more global
region-based approaches, and further with integration of shape priors.

Tract-based Statistical Analysis
================================

Summary
-------

Diffusion tensor imaging has made it possible to evaluate the
organization and coherence of white matter fiber tracts. Hence, it has
been used in many population studies, most notably, to find
abnormalities in schizophrenia. To date, most population studies
analyzing fiber tracts have used a single tensor as the local fiber
model. While robust, this model is known to be a poor fit in regions of
crossing or branching pathways. Nevertheless, the effect of using better
alternative models on population studies has not been studied. The goal
of this section is to compare white matter abnormalities as revealed by
two-tensor and single-tensor models. To this end, we compare three
different regions of the brain from two populations: schizophrenics and
normal controls. Preliminary results demonstrate that regions with
significant statistical difference indicated using one-tensor model do
not necessarily match those using the two-tensor model and vice-versa.
We demonstrate this effect using various tensor measures. This chapter
follows our work in @Malcolm2009study.

Introduction
------------

Diffusion tensor imaging (DTI) has become an established tool for
investigating tissue structure, and many studies have used it to
understand the effects of aging or disease. Using this imaging
technique, neuroscientists wish to ask how regions of tissue compare or
how well-defined various connections may be. For example, several DTI
studies have indicated a disturbance in connectivity between different
brain regions, rather than abnormalities within any specific region as
responsible for the cognitive dysfunctions observed in schizophrenia
@Kubicki2007. For such studies, the quality of the results depends on
the accuracy of the underlying model.

The most common local fiber model used in population studies is a single
diffusion tensor which provides a Gaussian estimate of diffusion
orientation and strength. While robust, this model can be inadequate in
cases of mixed fiber presence or more complex orientations, and so
various alternatives have been introduced including mixture models
@Alexander2001 [@Tuch2002; @Peled2006; @Hall2008] as well as
nonparametric approaches @Tuch2002
[@Descoteaux2007; @Jansons2003; @Tournier2004; @Jian2007ni].
Probabilistic techniques have also been developed in connectivity
studies @Parker2003 [@Behrens2007].

Despite this wide selection of available models, nearly all population
studies thus far have been based on the single-tensor model, and as such
it is important to examine the limits of this model and potential impact
this has. To start, some works have focused on the effects of noise and
acquisition schemes and have found nontrivial effects on estimated
fractional anisotropy (FA) and other quantities @Basser2000noise
[@Jones2004; @Goodlett2007]. Beyond such factors, it is known that in
regions containing crossing or fanning pathways the single-tensor model
itself provides a poor fit that results in lower FA @Alexander2002
[@Tuch2002]. It is estimated that as much as one third of white matter
may contain such putative fiber populations @Behrens2007. It is here
that this present study focuses.

In forming studies, there are several approaches for comparing patient
populations. For example, voxel-based studies examine tissue
characteristics in regions of interest @Ashburner2000. In contrast,
tract-based studies incorporate the results of tractography to use fiber
pathways as the frame of reference @Ding2003 [@Smith2006], and several
studies have demonstrated the importance of taking into account local
fluctuations in estimated diffusion @Corouge2006
[@Fletcher2007ipmi; @ODonnell2007; @Maddah2008; @Goodlett2009].

To date, many studies have focused on schizophrenia, but the findings
vary. For example, the review by Kubicki <span>et al</span>@Kubicki2007
cites one voxel-based study where the genu of the corpus callosum has
significant differences and another that finds nothing. A recent
tract-based study showed statistical differences not only in the genu of
the corpus callosum but also in regions known to have fiber crossings
and branchings @Maddah2008. The question then arises, as to whether the
same effect can be seen by better models able to resolve crossings and
branchings? Is the population difference simply a result of poor
modeling? We seek to answer such questions in this present work.

Our contributions
-----------------

In this chapter, we make a first attempt towards confirming or creating
doubts regarding the results reported using the single-tensor model.
Specifically, we compare various tensor metrics as estimated using
two-tensor and single-tensor models on a number of fiber tracts
generated using our recently proposed method for deterministic
two-tensor tractography @Malcolm2009ipmi. Hence we are continuing this
work with a focus on how such techniques will begin affecting clinical
studies. It is important to note that this is the first time the same
population study has been performed using two different underlying
models.

We begin with synthetic experiments to examine the difference in
reported FA using single- and two-tensor models. We find that the
single-tensor model consistently underestimates the FA by as much as 30%
in crossing regions, a difference considered statistically significant
in many studies. Then, we look at connections between three different
cortical regions of the brain and show that statistical group
differences reported using the single-tensor model do not necessarily
show differences using the two-tensor model and vice-versa.
Specifically, the regions known to have branchings and crossings reports
significant differences in the single tensor study, but not in the
two-tensor study, and conversely, certain regions which show subtle
abnormalities using the two-tensor method are lost in the single-tensor
model. Thus, model error may have contributed to the statistical
differences found in previous DTI studies.

Method {#sec:method}
------

In this chapter, we form a tract-based study using the two-tensor
tractography method described in , and we compare this against the
results from a single-tensor model typically used in such studies.
provides the necessary background on modeling the measured signal using
tensors and defines the specific two-fiber model employed in this study.
looks at the seed regions and the resulting fibers connecting each
hemisphere and finally how these fibers are compared within a
tract-based coordinate system.

Several scalar measures have been proposed for quantifying various
aspects of tensors, and in this study we focus on fractional anisotropy
(FA), trace, and the ratio between eigenvalues ($\lambda_2/\lambda_1$).
Since these measures are defined for the single-tensor model, when
computing their value on the two-tensor model, we record the value from
the tensor most aligned with the local fiber tangent.

Tractography and fiber comparison {#sec:fibers}
---------------------------------

[fig:roi~f~ibers]

For each patient, we have manually delineated cortical regions from
which we choose three regions covering the frontal and parietal lobe.
For one patient, shows these seed regions and the resulting fibers that
connect each hemisphere. Specifically, the regions are the
<span><span>*superiorfrontal*</span></span>,
<span><span>*precentral*</span></span>, and <span><span>*caudalmiddle
frontal*</span></span>.

We followed the deterministic fiber tracking procedure in
@Malcolm2009ipmi. We begin by seeding each algorithm several times in
each voxel of the seed regions. To explore branchings found using the
proposed technique, we considered a component to be branching if it was
separated from the primary component by less than
40<span>$^\circ$</span>with FA$\ge$0.15. We terminated fibers when
either the general fractional anisotropy @Tuch2002 of the estimated
signal fell below 0.1 or the primary component FA fell below 0.15. While
such parameters are heuristic in nature and could be examined in their
own right, we found the resulting tractography to be sufficient for the
purposes of this work.

[fig:ds~1~T]

It is known that single-tensor streamline tractography is only able to
trace out the dominant pathways forming the U-shaped callosal radiation,
so previous tract-based studies looking at the corpus callosum have been
restricted to only studying portions of the corpus radiata, typically
focusing on the splenum and genu @Corouge2006
[@Fletcher2007ipmi; @Goodlett2008; @Maddah2008]. One of the main
advantages of using the multi-tensor filtering approach in
@Malcolm2009ipmi is that it is one of the few techniques capable of
following, not only these dominate pathways, but also the transcallosal
pathways out to the lateral gyri. For example, shows that for the
<span><span>*caudalmiddle frontal*</span></span>region, using
single-tensor tractography leads to nearly all fibers looping back into
another sulci instead of finding any connection to the opposite
hemisphere. In contrast, demonstrates that the filtered two-tensor
approach finds several connecting pathways. Therefore, this is the first
attempt at tract-oriented analysis along such pathways.

[fig:fa]

Since this study focuses on comparing fiber models, we simply perform
direct single-tensor estimation at the same locations as we have
two-tensor estimates, thus providing an exact correspondence for
averaging within each patient. shows the
<span><span>*precentral*</span></span>region and colors the same fibers
with FA intensity to show the relative FA differences reported by either
model.

After performing tractography, we placed fibers from each patient within
a common coordinate system by registering the seed regions to a
template. Each seed region was registered separately. The mid-sagittal
plane was automatically determined and provided a common reference point
for each fiber as it passed through the corpus callosum. From this
reference point, we record arc-length moving outward to the cortical
regions. The fibers of each patient being registered with a template and
having an origin, we can plot the various scalar tensor measures along
this arc-length as in other tract-based studies. For example, shows FA
as a function of arc-length for the <span><span>*caudalmiddle
frontal*</span></span>region using the single-tensor model *(blue)* and
two-tensor model *(red)*. Since the single- and two-tensor estimates
line up exactly, any correspondence error is confined to the matching
within fiber bundles and among patients–not across models which was our
focus here. As in @ODonnell2007 [@Maddah2008], statistical significance
was tested as a function of arc-length.

Results
-------

We first use experiments with synthetic data to validate our technique
against ground truth. By varying crossing angle or eigenvalues used to
construct voxels, we confirm that our approach accurately estimates the
fractional anisotropy while using the single-tensor model can
under-estimate FA by as much as 30-40%. (). Then, we examine our real
dataset to demonstrate the different results reported using either model
().

### Synthetic validation {#sec:synthetic}

[fig:study~s~ynthetic]

Following the experimental method of generating synthetic data found in
@Tournier2004 [@Descoteaux2007], we generated synthetic MR signals
according to using eigenvalues determined from out *in vivo* data. We
use 81 gradient directions uniformly spread on the hemisphere and Rician
noise (<span>SNR $\approx$ 10 dB</span>) based on the unweighted signal.
Using these we constructed a set of two-dimensional fields through which
to navigate while estimating FA.

shows the resulting FA estimates using direct single-tensor estimation
*(blue)* and filtered two-tensor estimation *(red)*. As expected,
demonstrates that crossing regions can lead to single-tensor FA
estimates as much as 0.3 lower than expected. In we look at both
techniques accurately estimating the FA of a single tensor as we adjust
the second eigenvalue ($\lambda_2$ in ). However, and [fig:theta90]
demonstrate a consistent drop in FA under the same range of eigenvalues.
This experiment demonstrates that we may expect as much as 0.2 to 0.3
drop in FA in regions of crossing and branching.

### *In vivo* model comparison {#sec:study_real}

. [fig:study]

We tested our approach on a human brain scans using a 3-Tesla magnet to
collect 51 diffusion weighted images on the hemisphere at a voxel size
of $1.66\times1.66\times1.7\,\text{mm}^3$ and with
$b=900\,\text{s/mm}^2$ in addition to eight baseline scans. Included in
this study are 17 normal controls and 22 schizophrenics; however, since
not connecting fibers were not found in all patients for all regions,
not all patients were represented in each regional group. Below are the
sizes for each region and group.

l@

c@

c@

c

& <span><span>*Caudalmiddle frontal*</span></span>&
<span><span>*Precentral*</span></span>&
<span><span>*Superiorfrontal*</span></span>\
Normals & 16 & 17 & 8\
Schizophrenics & 20 & 22 & 17

For each region and patient group, shows the resulting average curves
using the single-tensor *(blue)* and two-tensor *(red)* models. Along
the bottom of each plot we indicate local regions of statistical
significance between groups. In we see that the two-tensor model
detected a region of significance across all three measures whereas the
single-tensor model found only one small portion of that in the trace.
As indicates, this a region known to contain branching and crossing,
hence the single-tensor FA drop. Thus, we suspect that the single-tensor
was unable to provide a tight enough fit in this region to detect the
difference found using the two-tensor model. In we see that the
single-tensor model found a slight area again in a region of known
branching, yet the two-tensor model found nothing. In we see the most
reported differences and further we see those differences reported using
both models. We note that these differences may in part be due to the
relative size of each population supporting this region. In summary,
among these three regions, we see areas where each model either
confirmed or denied the findings of the other.

Conclusion
----------

There are many challenges in building automatic frameworks for detecting
population differences. By repeating the same tract-based study and
changing only the model, we have demonstrated that the ultimate findings
may vary. Specifically, our results indicate that some areas reported as
significant using the single-tensor model may in fact be due to poor
modeling at branchings or crossings.

While our results are preliminary, we believe that exploring both
alternative models and methods of reconstructing pathways will provide
more accurate and comprehensive information about neural pathways and
ultimately enhance non-invasive diagnosis and treatment of brain
disease.

Considering future work, we expect that further model discrepancies may
be revealed with more accurate fiber and patient correspondences
@ODonnell2007 [@Maddah2008] or functional representations @Goodlett2008.

Summary
=======

Studies involving deterministic tractography rely on the underlying
model estimated at each voxel as well as the reconstructed pathways.
Most of the work on deterministic and probabilistic tractography has
involved estimating the fiber-ODF independently at each voxel and
performing tractography as a post-processing step with path
regularization. In this work, we proposed a method for simultaneous
estimation and tractography using two- and three-tensor mixtures. Using
an unscented Kalman filter provided robust parameter estimation and
demonstrated significantly higher angular accuracy compared to various
nonparametric and independent optimization techniques. Specifically we
found an angular error of 5-10<span>$^\circ$</span>in regions with fiber
crossings compared to 15-20<span>$^\circ$</span>using a common spherical
harmonic technique, and it is able to reliably resolve crossings down to
20-30<span>$^\circ$</span>compared to spherical harmonics which reaches
only down to 50-60<span>$^\circ$</span>. <span>*In
vivo*</span>experiments demonstrate the ability of the proposed method
to reveal fibers known to exist anatomically, <span>*e.g*</span>lateral
transcallosal fibers or temporal insertions along the fronto-occipital
fasciculus, yet absent using the comparison techniques.

Nevertheless, there are several aspects of this technique that could be
improved in future work. First, we believe improvements can certainly be
made with the local model. For example, switching filters may provide a
natural method for reduced parameter estimation where appropriate.
Further, we believe it is still important to explore constrained models
(convex weights, positive eigenvalues) @malcolm2009cukf as well as
approaches at model selection @Behrens2007.

Second, as is often the case in tractography, reported connections may
be invalid (false-positives). For example, the partial voluming along
the cingulum bundle in causes many spurious traces. There are several
possible approaches to suppressing such false positives
(<span>*e.g*</span>higher resolution scans, more appropriate local
models, alternative filter formulations), but we believe that
incorporating global information will ultimately have more effect than
more local choices in the model or filter.

In summary, we believe that filtering techniques offer significant
increases in sensitivity over traditional independent optimization
methods, but care must be taken ensuring anatomically correct results.
We believe that exploring both alternative models and filtering
techniques will provide more accurate and comprehensive information
about neural pathways and ultimately enhance non-invasive diagnosis and
treatment of brain disease.
