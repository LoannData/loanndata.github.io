---
permalink: /_portfolio/crspectra/
title: "A Cosmic Rays Self Propagation Escaping and Confinment TRAnsport code (CR SPECTRA)"
classes: wide
---
[Github](https://github.com/LoannData/CR_SPECTRA){: .btn .btn--primary}

The Cosmic Rays Self Propagation Escaping and Confinement TRAnsport code (CR SPECTRA) describe the non-linear relation between the way high energy 
particles are released in the Interstellar Medium from Supernova Remnant and the magnetic turbulence they generate in a close environment around the 
source, turbulence which contributes to confine particles close to the accelerator and then to affect the escape process itself. 

The code consists in solving the following non-linearly coupled system of four equations 

$$ \begin{aligned}
\frac{\partial P_\mathrm{p}}{\partial t} + V_\mathrm{A} \frac{\partial P_\mathrm{p}}{\partial z} & = 
\frac{\partial}{\partial z} \left( \kappa_{zz} \frac{\partial P_\mathrm{p}}{\partial z} \right) +
\frac{E}{3} \left\{ V_\mathrm{A}, P_\mathrm{p} \right\} - 
\frac{4}{3} \frac{\partial V_\mathrm{A}}{\partial z} P_\mathrm{p} + Q_{\mathrm{p}, \mathrm{inj}} \\ 
\frac{\partial P_\mathrm{e}}{\partial t} + V_\mathrm{A} \frac{\partial P_\mathrm{e}}{\partial z} & = 
\frac{\partial}{\partial z} \left( \kappa_{zz} \frac{\partial P_\mathrm{e}}{\partial z} \right) +
\frac{E}{3} \left\{ V_\mathrm{A}, P_\mathrm{e} \right\} - 
\frac{4}{3} \frac{\partial V_\mathrm{A}}{\partial z} P_\mathrm{e} + Q_{\mathrm{p}, \mathrm{inj}} + Q_{\mathrm{e}, \mathrm{sync}}(P_\mathrm{e}) \\ 
\frac{\partial I_+}{\partial t} + V_\mathrm{A} \frac{\partial I_+}{\partial z} & = 
- I_+ \frac{\partial V_\mathrm{A}}{\partial z} + \Gamma_\mathrm{g}^+ I_+ - 
\Gamma_\mathrm{d}^+ (I_+ - I_+^0) \\ 
\frac{\partial I_-}{\partial t} + V_\mathrm{A} \frac{\partial I_-}{\partial z} & = 
- I_- \frac{\partial V_\mathrm{A}}{\partial z} + \Gamma_\mathrm{g}^- I_- - 
\Gamma_\mathrm{d}^+ (I_- - I_-^0) 
\end{aligned} $$ 

in a two dimensional phase space where the first dimension represents the spatial direction in which galactic scale magnetic field lines are 
correlated (see the figure above) and the second dimension represents the energy associated to the resonant Alfvén waves interacting with the released CRs. 

The system is solved using first and second order finite difference methods for explicit solvers and implicit methods for the diffusion term. This system has 
some particularities: 

- The diffusion coefficient $$\kappa_{zz}$$ depends on the turbulence rates $$I_\pm$$ 
- An energy log scale has been used to solve energy related terms 
- The Interstellar Medium model is inhomogeneous leading to a space variable initial diffusion coefficient $$\kappa_{zz}(z, t = 0)$$
- Cosmic Rays (protons p and electrons e) are injected through a term $$Q_{\mathrm{p,e,inj}}$$ itself depending on the turbulence rates $$I_\pm$$ 
- Cosmic Rays generate turbulence $$I_\pm$$ through linear thermal instability terms $$\Gamma_g^\pm$$ 
- CRs generated turbulence $$I_\pm$$ is progressively damped ($$\Gamma_\mathrm{d}^\pm$$) down to the background turbulence state $$I_\pm^0$$ according the Interstellar Medium plasma physical properties

{% include video id="H-M5yKDYbd0" provider="youtube" %}

This video represents a simulation of my CRs injection and propagation model. More informations [here](https://ui.adsabs.harvard.edu/abs/2020A%26A...633A..72B/abstract)