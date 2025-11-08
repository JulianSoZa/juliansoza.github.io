---
title: An overview of Geological Lineament Detection Using Gravity and Magnetic Fields
date: 2025-08-19
math: true
authors:
  - name: JulianSoZa
    link: https://github.com/JulianSoZa
    image: https://github.com/JulianSoZa.png
category: Lineament Detection
tags:
  - Geophysics
  - Lineaments
  - Potential
  - Theory
  - Overview
excludeSearch: false
---
In this post, I provide an overview of the role of gravity and magnetic fields in geological lineament detection, based on several bibliographic resources I have studied.
<!--more-->

## What is a geological lineament?

Lineaments are linear geological structures identified in geophysical images. Their detection involves interpreting a wide range of geoscientific data, such as topographic, magnetic, and gravity raster data, to delineate geological structures and define high-priority areas for further exploration.

The structures delineated in geophysical data are typically associated with geological structures in the Earth's crust, such as subsurface faults, fractures, shear zones, and contacts. Recognizing these structures has important applications, including groundwater exploration, mineral resource exploration, and the assessment of landslides and earthquakes [^1]. The location of a lineament can be identified in geophysical data by changes in gradient magnitude, variations in pattern, or the occurrence of linear minima or maxima [^1].

## Why are gravity and magnetic data useful?

The acceleration due to gravity is influenced by topography, aspherical variation of density within the Earth, and the Earth’s rotation [^2], gravity measurements can therefore be used to identify density variations caused by geological structures. Similarly, measurements of the Earth’s magnetic field can reveal magnetic anomalies associated with subsurface structures; some of these anomalies may indicate crustal features, such as mineralized dykes or other structures of commercial interest [^3].

Observations of the Earth's potential fields are compared with their theoretical values, which are based on reference geometries: the Reference Ellipsoid for the gravitational field and the International Geomagnetic Reference Field (IGRF) for the magnetic field. Differences between observed and theoretical values are referred to as gravitational or magnetic anomalies, depending on the case.

### Reduction of gravity and magnetic data through corrections and transformations

Due to topographical variations in the Earth's surface, the location where measurements are taken influences the results obtained and the corrections that must be made. Gravitational field measurements must be corrected for the effect of elevation above sea level (Free Air correction), the topography of the environment such as valleys and mountains (Bouguer and terrain corrections), and the effect of masses supporting topographic loads (isostatic correction) [^2] [^4].

In magnetic field measurements, differences in the orientation between the Earth's main magnetic field and the magnetic source can cause a skewness of the anomalies [^5]. This effect can be corrected by applying the reduction to pole (RTP) or reduction to equator (RTE) transforms, depending on the geomagnetic inclination of the fields [^6].

### Gravity anomalies

A gravity anomaly is the deviation of a gravity measurement from the normal gravity value defined by the Reference Ellipsoid at the same location. Gravimeters measure gravity anomalies at the microgal scale ($1 \ \mathrm{\mu Gal} = 1 \times 10^{-8} \ \mathrm{m/s^2}$) [^4].

Depending on the corrections applied, different types of gravity anomalies are defined:

#### The free-air anomaly

This anomaly occurs when gravity is measured at the Earth's surface at a given altitude and only the free-air correction is applied. The anomaly is calculated by subtracting the correction for elevation above sea level ($dg_{FA}$) and the reference gravity field ($g_0$) from the measured gravity ($g_{obs}$) [^2]:

$$\Delta g_{FA} = g_{obs} - dg_{FA} - g_0.$$

#### The Bouguer anomaly

The free-air correction does not account for any attracting mass between the observation point and sea level, since it assumes no mass [^2]. At a given elevation in the Earth's surface, such as a mountain where the measurement is made, the mass of the terrain exerts an attraction on the gravimeter that must be removed. This mass is replaced by a uniform slab between the gravimeter and the ellipsoid, known as the "slab approximation", which assumes the rocks extend infinitely in the horizontal direction [^2] [^4]. Calculating this anomaly requires knowledge of the local rock density, and finally, the anomaly is given by:

$$\Delta g_{B} = g_{obs} - dg_{FA} - g_0-dg_B.$$

Any topography higher than the gravimeter has a component of attraction, and a nearby valley represents missing material which also weakens the local gravity. Therefore, a terrain correction must be applied for mountains and valleys surrounding the measurement station [^4].

#### The isostatic anomaly

In an isostatic equilibrium (similar to an iceberg floating in water) a mountain in the Earth's crust requires a compensating root that is less dense than the surrounding rock at the same depth, in this process, crustal material replaces denser mantle material, following Archimedes' Law [^2]. The isostatic correction is applied to remove the effect of masses supporting topographic loads (the crustal root), so that the isostatic anomaly isolates the small signal due to the uncompensated density anomaly [^2].

### Magnetic anomalies

A magnetic anomaly is the deviation of the measured field from the theoretical value expected at a given location (as defined by the IGRF). This deviation arises from inhomogeneities in subsurface magnetization [^3]. Precession magnetometers measure only the magnitude of the total magnetic field, not its direction, and have a sensitivity of about $1 \ \mathrm{nT}$ [^5].

The total field $\textbf{B}$ is the contribution of the Earth's main field $\textbf{B}_E$ and the magnetic source $\delta \textbf{B}$. However, scalar magnetometers measure only the magnitude of this field. Thus, a magnetic anomaly $A$ is given by

$$A = B - B_E = \hat{\textbf{B}}_E \cdot \delta \textbf{B},$$

where only the component of the local field $\delta \textbf{B}$ parallel to the Earth's main field is measured [^5].

## How can we model gravity and magnetic fields?

### Potential Field Theory

Potential field theory is used to mathematically model the Earth's magnetic and gravitational fields. These fields are described as the gradient of a potential $V(\textbf{r})$, to which Poisson's (1) and Laplace's (2) equations are applied. Both fields are dominated by simple geometries used as theoretical references for processing observations, but higher-order components are necessary to accurately model their true shapes with greater detail [^5].


$$
\begin{equation}
  \nabla^2 V(\textbf{r})=\rho(\textbf{r}),  \quad \text{Poisson's equation}
\end{equation}
$$

$$
\begin{equation}
      \nabla^2 V(\textbf{r})=0,  \quad \text{Laplace's equation}
\end{equation}
$$

The general solution to Laplace's equation in spherical coordinates for an external field in the space outside a sphere of radius $R$ with center at the coordinate origin is

$$
V(\theta, \lambda, r) = \sum_{n=0}^{\infty} \sum_{m=-n}^{n} \left( \frac{R}{r} \right)^{n+1} v_{nm} \, \overline{Y}_{nm}(\theta, \lambda),
$$

where $\overline{Y}_{nm}(\theta, \lambda)$ are the spherical harmonics defined as

$$
\overline{Y}_{nm}(\theta, \lambda) = \overline{P}_{n|m|}(\cos\theta) \cdot
\begin{cases}
\cos m\lambda, & m \geq 0 \\
\sin |m|\lambda, & m < 0
\end{cases}
$$
and $\overline{P}_{n|m|}$ is a normalization of the associated Legendre function of the first kind [^7].
### Gravity field

Gravity is the effect exerted on a mass $m$ due to the combined influence of the Earth's gravitational attraction (with mass $M$) and the Earth's rotation. The gravitational acceleration or gravity field $\mathbf{g}$ is the force per unit mass due to gravity, and it is also the gradient of the gravitational potential $U$, which represents the potential energy per unit mass in the field created by $M$ [^2]. If $M$ is a point mass, we have:


$$
\mathbf{g} = -\frac{GM}{r^{2}} \, \hat{\mathbf{r}} 
= \frac{\partial}{\partial r} \left( \frac{GM}{r} \right) 
= -\frac{\partial}{\partial r} U 
= -\nabla U,
$$
where, $G$ is the universal gravitational constant. However, due to the actual shape of the Earth and its rotation, the gravitational potential is more complex, and the rotational potential must be added:

$$ 
U_{\text{gravity}} = U_{\text{gravitation}} + U_{\text{rot}}.
$$

The Reference Ellipsoid is an equipotential surface where the potential is constant along the surface, but the gravity itself is not, and the potential value is defined to be the same as that of mean sea level [^4]. The mathematical expression for the gravitational potential component $V^e$ on the ellipsoid, using the general solution of Laplace's equation, is given by

$$
V^e(\theta, \lambda, r) = \frac{GM}{r} \left( 1 - \sum_{n=1}^{\infty} \mathcal{J}_{2n} \left( \frac{a}{r} \right)^{2n} P_{2n}(\cos\theta) \right),
$$
where $\mathcal{J}_{2n}$ is the zonal expansion coefficient, and $a$ is the semimajor axis of the ellipsoid [^7].
### Magnetic field

Moving electric charges (such as electric currents) and magnetic bodies produce a magnetic field that describes the magnetic influence of these sources. By analogy with the electric or gravitational potential (the work done per unit charge or mass) the magnetic potential $V$ for the magnetic flux density $\textbf{B}$ (ambiguously called the magnetic field) is the work done per unit current, as indicated by its units $\left[\mathrm{kg\cdot m^2/s^2\cdot A}\right]$. The increment in magnetic potential [^5] is defined as

$$
dV = -\mathbf{B} \cdot d\mathbf{l} 
\iff 
\mathbf{B} = -\nabla V
$$

About 90% of the magnetic field at the Earth’s surface is a dipole inclined at approximately 11° to the Earth’s spin axis, the remaining 10% is known as the non-dipole field (NDF) [^3]. The IGRF is the reference magnetic field, represented by a series of mathematical models of the Earth’s main field, it is updated periodically (usually every four years) to account for secular variations due to its dynamic nature. The [mathematical model](https://www.ncei.noaa.gov/products/international-geomagnetic-reference-field), based on the solution to Laplace's equation, is given by

$$
V(r, \theta, \lambda, t) = a \sum_{n=1}^{N} \sum_{m=0}^{n} \left( \frac{a}{r} \right)^{n+1} 
\left[ g_n^m(t) \cos(m\lambda) + h_n^m(t) \sin(m\lambda) \right] P_n^m(\cos\theta)
$$
and the degree of truncation is N = 13 [^8].
## The *Fatiando a Terra* project - Python libraries

*Fatiando* [^9] is a collection of **open-source Python packages** for data processing, modeling, and inversion, primarily aimed at geophysics.

For example, the [Harmonica](https://www.fatiando.org/harmonica/latest/) library includes processing steps such as Bouguer and terrain corrections, reduction to the pole, and more. It provides forward modeling functions for basic geometric shapes, including point sources, prisms, and tesseroids, as well as inversion methods. With Harmonica, we can compute the gravitational effect of topography from a terrain model.

## Algorithms for the extraction of geological lineaments

There are various methods to delineate geological structures from geoscientific data. In [^10], the authors use a classical algorithm that calculates the curvature of a surface to determine whether a specific coordinate is at a minimum. Authors in [^1] use a deep learning approach, a convolutional neuronal network (CNN). The output value is a probability value that corresponds to existence of lineament, they can make a probability map for any region using their trained parameters. The model is trained with a probability map that corresponds to an existing lineament, this map was created based on data interpreted by experts. As a post-processing method, they convert probability maps into lines and polynomials using clustering algorithms.

## References

[^1]: A. Aghaee, P. Shamsipour, S. Hood, and R. Haugaard, “A convolutional neural network for semi-automated lineament detection and vectorisation of remote sensing data using probabilistic clustering: A method and a challenge,” Comput. Geosci., vol. 151, p. 104724, June 2021, doi: [10.1016/j.cageo.2021.104724](https://doi.org/10.1016/j.cageo.2021.104724).

[^2]: MIT OpenCourseWare. “Chapter 2: The Earth’s Gravitational field," *Essentials of Geophysics*. Massachusetts Institute of Technology. [Online]. Available: [ocw.mit.edu](https://ocw.mit.edu/courses/12-201-essentials-of-geophysics-fall-2004/resources/ch3pdf/).

[^3]: W. Lowrie, “The Earth’s magnetic field,” in _Geophysics: A Very Short Introduction_, W. Lowrie, Ed., Oxford University Press, 2018. doi: [10.1093/actrade/9780198792956.003.0007](https://doi.org/10.1093/actrade/9780198792956.003.0007).

[^4]: W. Lowrie, “Gravity and the figure of the Earth,” in _Geophysics: A Very Short Introduction_, W. Lowrie, Ed., Oxford University Press, 2018. doi: [10.1093/actrade/9780198792956.003.0005](https://doi.org/10.1093/actrade/9780198792956.003.0005).

[^5]: MIT OpenCourseWare. “Chapter 3: The Magnetic Field of the Earth," *Essentials of Geophysics*. Massachusetts Institute of Technology. [Online]. Available: [ocw.mit.edu](https://ocw.mit.edu/courses/12-201-essentials-of-geophysics-fall-2004/resources/ch3pdf/).

[^6]: C. Foss, “Magnetic Data Enhancements and Depth Estimation,” in _Encyclopedia of Solid Earth Geophysics_, H. K. Gupta, Ed., Dordrecht: Springer Netherlands, 2011, pp. 736–746. doi: [10.1007/978-90-481-8702-7_104](https://doi.org/10.1007/978-90-481-8702-7).

[^7]: C. Jekeli, “3.02 - Potential Theory and the Static Gravity Field of the Earth,” in _Treatise on Geophysics (Second Edition)_, G. Schubert, Ed., Oxford: Elsevier, 2015, pp. 9–35. doi: [10.1016/B978-0-444-53802-4.00056-7](https://doi.org/10.1016/B978-0-444-53802-4.00056-7).

[^8]: P. Alken _et al._, “International Geomagnetic Reference Field: the thirteenth generation,” _Earth Planets Space_, vol. 73, no. 1, p. 49, Feb. 2021, doi: [10.1186/s40623-020-01288-x](https://doi.org/10.1186/s40623-020-01288-x).

[^9]: L. Uieda, V. C. O. Jr, and V. C. F. Barbosa, “Modeling the Earth with Fatiando a Terra,” _scipy_, May 2013, doi: [10.25080/Majora-8b375195-010](https://doi.org/10.25080/Majora-8b375195-010).

[^10]: M. Lee, W. Morris, J. Harris, and G. Leblanc, “An automatic network-extraction algorithm applied to magnetic survey data for the identification and extraction of geologic lineaments,” Lead. Edge, vol. 31, no. 1, pp. 26–31, Jan. 2012, doi: [10.1190/1.3679324](https://doi.org/10.1190/1.3679324).