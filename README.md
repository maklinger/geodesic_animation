# geodesic_animation
A Kerr BH with photon geodesics coming in. Can be used as a screen saver. Created using Claude.

# Photon geodesics around a spinning black hole

**Live demo:** [maklinger.github.io/geodesic_animation](https://maklinger.github.io/geodesic_animation/)

An interactive, real-time animation of photon trajectories in the equatorial plane of a Kerr black hole. Photons are integrated along null geodesics using a 4th-order Runge–Kutta scheme with adaptive subcycling near the black hole. Colour encodes gravitational redshift and blueshift relative to each photon's starting radius. A parallel-beam mode lets you fire a planar wavefront at the black hole and watch gravitational lensing and frame dragging bend the rays.

---

## Physics

### Coordinates

The simulation uses **Boyer–Lindquist coordinates** $(t, r, \theta, \phi)$, restricted to the equatorial plane $\theta = \pi/2$. The black hole has mass $M$ and spin parameter $a$ (with $0 \le a < M$). We work in geometrized units where $G = c = 1$, so the gravitational radius is $r_g = M$.

The Kerr metric in the equatorial plane is:

$$ds^2 = -\left(1 - \frac{r_s r}{r^2}\right)dt^2 - \frac{2 r_s r a}{r^2} dt d\phi + \frac{r^2}{\Delta} dr^2 + \frac{A}{r^2} d\phi^2$$

where $r_s = 2M = 2r_g$ and:

$$\Delta = r^2 - r_s r + a^2$$

$$A = (r^2 + a^2)^2 - a^2 \Delta$$

The **event horizon** is at $r_+ = M + \sqrt{M^2 - a^2}$, and the **ergosphere** (equatorial) at $r_\mathrm{ergo} = r_s = 2r_g$.

---

### Geodesic equation and conserved quantities

A geodesic satisfies $\nabla_u u^\mu = 0$, where $u^\mu = dx^\mu/d\lambda$ and $\lambda$ is an affine parameter. For a photon the geodesic is null:

$$g_{\mu\nu} u^\mu u^\nu = 0$$

The Kerr metric is stationary and axisymmetric, so two quantities are conserved along any geodesic:

$$E = -g_{t\mu} u^\mu \quad \text{(energy per unit mass)}$$

$$L = g_{\phi\mu} u^\mu \quad \text{(angular momentum per unit mass)}$$

The ratio $b = L/E$ is the **impact parameter** — the effective perpendicular distance of the ray from the black hole at infinity.

---

### Equations of motion

Define:

$$P = E(r^2 + a^2) - aL$$

$$\mathcal{R} = P^2 - \Delta (L - aE)^2$$

The angular equation is:

$$\dot{\phi} = \frac{L}{r^2} - \frac{a(aL - (r^2+a^2)E)}{r^2 \Delta}$$

The radial equation follows from the null condition:

$$r^4 \dot{r}^2 = \mathcal{R}$$

so $p_r \equiv \dot{r} = \pm\sqrt{\mathcal{R}}/r^2$. Differentiating with respect to $\lambda$:

$$\dot{p}_r = \frac{d\mathcal{R}/dr}{2r^4} - \frac{2p_r^2}{r}$$

where:

$$\frac{d\mathcal{R}}{dr} = 4PEr - (2r - r_s)(L - aE)^2$$

The full system that is integrated is:

$$\frac{dr}{d\lambda} = p_r$$

$$\frac{d\phi}{d\lambda} = \dot{\phi}(r, L, E)$$

$$\frac{dp_r}{d\lambda} = \dot{p}_r(r, p_r, L, E)$$

---

### Initialisation

Each photon is spawned at a starting position $(r_0, \phi_0)$ on the edge of the screen, converted from pixel coordinates to Boyer–Lindquist coordinates via a fixed scale factor.

**Random mode:** the impact parameter $b$ is drawn randomly near the critical value $b_\mathrm{crit} \approx 3\sqrt{3} M$, which corresponds to the unstable circular photon orbit (photon sphere). The initial radial momentum is set from the null condition:

$$p_{r,0} = -\sqrt{\mathcal{R}(r_0)} / r_0^2$$

(negative = inward).

**Parallel beam mode:** photons are placed at evenly spaced transverse offsets along the far edge of the screen, all moving in the same Cartesian direction $(\cos\alpha, \sin\alpha)$ for beam angle $\alpha$. The impact parameter for each ray is the signed perpendicular distance from the black hole to the ray:

$$b = x_0 \sin\alpha - y_0 \cos\alpha$$

where $(x_0, y_0)$ are the BL-frame Cartesian coordinates of the starting point. The sign of $p_{r,0}$ is negative (inward) when the beam direction has a negative radial component, i.e. when $\hat{b} \cdot \hat{r} < 0$.

---

### Integration

The system is integrated using a **4th-order Runge–Kutta** (RK4) scheme. Each call to the integrator advances the photon by one macro step of fixed affine size $\Delta\lambda$, but near the event horizon the macro step is transparently split into $n$ sub-steps of size $\Delta\lambda / n$:

$$n(r) = \mathrm{round}\left(1 + (n_\mathrm{max} - 1) \cdot \frac{r_\mathrm{thresh} - r}{r_\mathrm{thresh} - r_+}\right) \quad \text{for } r < r_\mathrm{thresh}$$

with $n = 1$ for $r \ge r_\mathrm{thresh}$. The default values are $n_\mathrm{max} = 8$ and $r_\mathrm{thresh} = 4\, r_+$. The total affine advance per macro step is always $\Delta\lambda$ regardless of $n$, so the visual speed of photons is unchanged. The sub-step positions are all appended to the trail, so the plotted curve is as smooth as the integration near the horizon.

Multiple macro steps are taken per rendered frame to decouple numerical accuracy from frame rate. A photon is removed when:

- $r < 1.02\, r_+$ — absorbed by the black hole
- $r > r_\mathrm{escape}$ — escaped beyond the screen diagonal

When a photon is absorbed, its trail is promoted to a **ghost**: integration stops, but the trail continues to be drawn and is trimmed from the oldest end at the same rate it was being extended, so it fades away over the same wall-clock time as a live trail of equal length.

In beam mode, absorbed or escaped photons are immediately respawned at their original position so the beam remains continuously full.

---

### Frame dragging visualisation

The ZAMO (zero angular momentum observer) angular velocity at radius $r$ in the equatorial plane is:

$$\Omega_\mathrm{ZAMO}(r) = \frac{r_s r a}{(r^2 + a^2)^2 - a^2 \Delta}$$

Small tracer dots are placed at fixed radii between the horizon and the ergosphere and advanced at this rate each frame, making the differential rotation of spacetime visible.

---

## Using as a macOS screensaver

The file `index.html` in this repository is fully self-contained — no server, no dependencies. You can run it as a screensaver using [WebViewScreenSaver](https://github.com/liquidx/webviewscreensaver).
