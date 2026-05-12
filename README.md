# geodesic_animation
A Kerr BH with photon geodesics coming in. Can be used as a screen saver. Made together with Claude.

# Photon geodesics around a spinning black hole

**Live demo:** [maklinger.github.io/geodesic_animation](https://maklinger.github.io/geodesic_animation/)

An interactive, real-time animation of photon trajectories in the equatorial plane of a spinning Kerr black hole. Photons are integrated along null geodesics using a 4th-order Runge–Kutta scheme. Colour encodes gravitational redshift and blueshift relative to each photon's starting radius. A parallel-beam mode lets you fire a planar wavefront at the black hole and watch gravitational lensing and frame dragging bend the rays.

---

## Physics

### Coordinates

The simulation uses **Boyer–Lindquist coordinates** $(t, r, \theta, \phi)$, restricted to the equatorial plane $\theta = \pi/2$. The black hole has mass $M$ and spin parameter $a$ (with $0 \le a < M$). We work in geometrized units where $G = c = 1$.

The Kerr metric in the equatorial plane is:

$$ds^2 = -\left(1 - \frac{r_s r}{\Sigma}\right)dt^2 - \frac{2 r_s r a}{\Sigma}\, dt\, d\phi + \frac{\Sigma}{\Delta}\, dr^2 + \frac{A}{\Sigma}\, d\phi^2$$

where $r_s = 2M$ is the Schwarzschild radius and in the equatorial plane ($\theta = \pi/2$, $\Sigma = r^2$):

$$\Delta = r^2 - r_s r + a^2, \qquad A = (r^2 + a^2)^2 - a^2 \Delta$$

The **event horizon** is at $r_+ = M + \sqrt{M^2 - a^2}$, and the **ergosphere** (equatorial) at $r_\text{ergo} = r_s = 2M$.

---

### Geodesic equation and conserved quantities

A geodesic satisfies $\nabla_u u^\mu = 0$, where $u^\mu = dx^\mu/d\lambda$ and $\lambda$ is an affine parameter. For a photon, the geodesic is null:

$$g_{\mu\nu} u^\mu u^\nu = 0$$

The Kerr metric is stationary and axisymmetric, so two quantities are conserved along any geodesic:

$$E = -g_{t\mu} u^\mu = \left(1 - \frac{r_s r}{r^2}\right)\dot{t} + \frac{r_s r a}{r^2}\dot{\phi} \quad \text{(energy)}$$

$$L = g_{\phi\mu} u^\mu = -\frac{r_s r a}{r^2}\dot{t} + \frac{A}{r^2}\dot{\phi} \quad \text{(angular momentum)}$$

The ratio $b = L/E$ is the **impact parameter** — the effective perpendicular distance of the ray from the black hole at infinity.

---

### Equations of motion

Inverting the conserved quantities and using the null condition gives the equations of motion for $(r, \phi)$. Define:

$$P = E(r^2 + a^2) - aL, \qquad \mathcal{R} = P^2 - \Delta\,(L - aE)^2$$

Then the angular velocity is:

$$\dot{\phi} = \frac{L}{r^2} - \frac{a\bigl(aL - (r^2+a^2)E\bigr)}{r^2 \Delta}$$

The radial equation follows from the null condition $g_{\mu\nu}u^\mu u^\nu = 0$:

$$r^4 \dot{r}^2 = \mathcal{R}$$

so $p_r \equiv \dot{r} = \pm\sqrt{\mathcal{R}}/r^2$. Differentiating with respect to $\lambda$:

$$\dot{p}_r = \frac{d\mathcal{R}/dr}{2r^4} - \frac{2p_r^2}{r}$$

where

$$\frac{d\mathcal{R}}{dr} = 2P \cdot 2Er - (2r - r_s)(L - aE)^2$$

The full system integrated is therefore:

$$\frac{dr}{d\lambda} = p_r, \qquad \frac{d\phi}{d\lambda} = \dot{\phi}(r, p_r, L, E), \qquad \frac{dp_r}{d\lambda} = \dot{p}_r(r, p_r, L, E)$$

---

### Initialisation

Each photon is spawned at a starting position $(r_0, \phi_0)$ on the edge of the screen, converted from pixel coordinates to Boyer–Lindquist coordinates via a fixed scale factor.

**Random mode:** the impact parameter $b$ is drawn randomly near the critical value $b_\text{crit} \approx 3\sqrt{3}\,M$, which corresponds to the unstable photon orbit (photon sphere). The initial radial momentum is set from the null condition:

$$p_{r,0} = -\sqrt{\mathcal{R}(r_0)}/r_0^2$$

(negative = inward).

**Parallel beam mode:** photons are placed at evenly spaced transverse offsets along the far edge of the screen, all moving in the same Cartesian direction $(\cos\alpha, \sin\alpha)$ for beam angle $\alpha$. The impact parameter for each ray is computed as the signed perpendicular distance from the black hole to the ray:

$$b = x_0 \sin\alpha - y_0 \cos\alpha$$

where $(x_0, y_0)$ are the BL-frame Cartesian coordinates of the starting point. The sign of $p_{r,0}$ is determined by whether the beam direction has an inward radial component: $p_{r,0} < 0$ iff $\hat{b} \cdot \hat{r} < 0$.

---

### Integration

The system is integrated using a **4th-order Runge–Kutta** (RK4) scheme with fixed step size $\Delta\lambda$. Multiple steps are taken per rendered frame to decouple numerical accuracy from visual speed. The visual speed of a photon scales as $\Delta\lambda \times \text{steps/frame} \times \text{fps}$.

A photon is removed when:
- $r < 1.02\, r_+$ — absorbed by the black hole
- $r > r_\text{escape}$ — escaped beyond the screen diagonal

In beam mode, absorbed or escaped photons are immediately respawned at their original position so the beam remains continuously full.

---

### Frame dragging visualisation

The **ZAMO** (zero angular momentum observer) angular velocity at radius $r$ in the equatorial plane is:

$$\Omega_\text{ZAMO}(r) = \frac{r_s r a}{(r^2 + a^2)^2 - a^2 \Delta}$$

Small tracer dots are placed at fixed radii between the horizon and the ergosphere and advanced at this rate each frame, making the differential rotation of spacetime visible.

---

## Using as a macOS screensaver

The file `static.html` in this repository is fully self-contained — no server, no dependencies. You can run it as a screensaver using [WebViewScreenSaver](https://github.com/liquidx/webviewscreensaver).

