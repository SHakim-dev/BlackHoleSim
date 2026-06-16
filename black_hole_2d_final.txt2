import sys
import math
import random
import numpy as np
import pygame

M  = 1.0
RS = 2.0 * M

R_PHOTON = 1.5 * RS
R_ISCO   = 3.0 * RS
R_DISK   = 5.0 * RS

WIDTH, HEIGHT = 950, 720
BG_COLOR = (2, 2, 14)

SCALE    = 40
ORIGIN_X = WIDTH  // 2
ORIGIN_Y = HEIGHT // 2

def world_to_screen(x, y):
    sx = int(ORIGIN_X + x * SCALE)
    sy = int(ORIGIN_Y - y * SCALE)
    return sx, sy

def geodesic_rhs(state, E, rs):
    # state = (r, phi, dr, dphi)
    r, phi, dr, dphi = state
    f = 1.0 - rs / r
    # dt/dlambda for null geodesic
    dt_dlambda = E / f

    rhs0 = dr
    rhs1 = dphi
    rhs2 = -(rs / (2.0 * r * r)) * f * (dt_dlambda * dt_dlambda) \
           + (rs / (2.0 * r * r * f)) * (dr * dr) \
           + (r - rs) * (dphi * dphi)
    rhs3 = -2.0 * dr * dphi / r
    return (rhs0, rhs1, rhs2, rhs3)


def rk4_step_state(state, dlam, E, rs):
    # standard RK4 for 4-component state
    k1 = geodesic_rhs(state, E, rs)
    y2 = tuple(state[i] + 0.5 * dlam * k1[i] for i in range(4))
    k2 = geodesic_rhs(y2, E, rs)
    y3 = tuple(state[i] + 0.5 * dlam * k2[i] for i in range(4))
    k3 = geodesic_rhs(y3, E, rs)
    y4 = tuple(state[i] + dlam * k3[i] for i in range(4))
    k4 = geodesic_rhs(y4, E, rs)
    new = tuple(state[i] + (dlam / 6.0) * (k1[i] + 2.0 * k2[i] + 2.0 * k3[i] + k4[i]) for i in range(4))
    return new

def trace_ray(b, dphi=0.004, max_steps=5000):
    # start point far on the left, at x = -r0, y = b
    r0 = 60.0
    x0 = -r0
    y0 = b
    r = math.hypot(x0, y0)
    phi = math.atan2(y0, x0)

    # initial direction: roughly to the right (1,0) in Cartesian
    dx = 1.0
    dy = 0.0

    # convert direction to polar basis (dr, dphi)
    dr = math.cos(phi) * dx + math.sin(phi) * dy
    dphi = (-math.sin(phi) * dx + math.cos(phi) * dy) / max(1e-12, r)

    # conserved quantities and normalization (null geodesic)
    L = r * r * dphi
    f = 1.0 - RS / r
    dt_dlambda = math.sqrt((dr * dr) / (f * f) + (r * r * dphi * dphi) / f)
    E = f * dt_dlambda

    state = (r, phi, dr, dphi)
    points = []

    dlam = 0.8
    for _ in range(max_steps):
        r, phi, dr, dphi = state
        if r <= RS:
            # captured
            x = r * math.cos(phi)
            y = r * math.sin(phi)
            points.append((x, y))
            return points, True

        x = r * math.cos(phi)
        y = r * math.sin(phi)
        points.append((x, y))

        # escaped to the right
        if len(points) > 10 and x > 0 and r > 50.0:
            return points, False

        if r > 120.0:
            return points, False

        # integrate forward in affine parameter
        state = rk4_step_state(state, dlam, E, RS)

    return points, False

def generate_rays(n_rays=100):
    b_crit = (3.0 * math.sqrt(3.0) / 2.0) * RS
    b_min = b_crit * 0.50
    b_max = b_crit * 4.5

    results = []
    for b in np.linspace(b_min, b_max, n_rays):
        pts, cap = trace_ray(float(b))
        results.append((pts, cap, float(b)))
    return results

def ray_color(b, captured):
    b_crit = (3.0 * math.sqrt(3.0) / 2.0) * RS
    t = (b - b_crit * 0.50) / (b_crit * 4.0)
    t = max(0.0, min(1.0, t))

    if captured:
        r  = 255
        g  = int(140 * (1.0 - t))
        bl = 0
    else:
        bright = 0.35 + 0.65 * t
        r  = int(20  * bright)
        g  = int(160 * bright)
        bl = int(255 * bright)

    return (r, g, bl)

def draw_filled_circle(surface, color, radius_phys):
    sx, sy = world_to_screen(0, 0)
    r_pix  = max(1, int(radius_phys * SCALE))
    pygame.draw.circle(surface, color, (sx, sy), r_pix)

def draw_circle_ring(surface, color, radius_phys, width=1):
    sx, sy = world_to_screen(0, 0)
    r_pix  = max(1, int(radius_phys * SCALE))
    pygame.draw.circle(surface, color, (sx, sy), r_pix, width)

def draw_ray(surface, points, color):
    if len(points) < 2:
        return
    screen_pts = [world_to_screen(x, y) for (x, y) in points]
    pygame.draw.lines(surface, color, False, screen_pts, 1)

def draw_ray_segment(surface, points, length, color):
    if length < 2:
        return
    draw_ray(surface, points[:length], color)

def draw_accretion_disk(surface):
    bh_sx, bh_sy = world_to_screen(0, 0)
    r_inner = int(R_ISCO * SCALE)
    r_outer = int(R_DISK * SCALE)
    n       = 50

    disk_surf = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
    for i in range(n):
        t = i / (n - 1)
        r  = r_inner + int(t * (r_outer - r_inner))
        rc = 255
        gc = int(210 - 160 * t)
        bc = int(60  -  60 * t)
        al = int(55  -  50 * t)
        pygame.draw.circle(disk_surf, (rc, gc, bc, max(1, al)),
                           (bh_sx, bh_sy), r, 3)
    surface.blit(disk_surf, (0, 0))

def draw_stars(surface, stars):
    for (x, y, b) in stars:
        surface.set_at((x, y), (b, b, b))

def draw_legend(surface, small_font):
    items = [
        ((10, 10, 10),     "Event horizon     r = rs = 2M"),
        ((200, 200, 255),  "Photon sphere     r = 3M  (unstable orbits)"),
        ((255, 160, 30),   "ISCO              r = 6M  (innermost stable orbit)"),
        ((255, 120, 0),    "Captured rays     b < b_crit  (spiral inward)"),
        ((30,  140, 255),  "Escaping rays     b > b_crit  (gravitational lensing)"),
    ]
    x0, y0 = 12, 12
    for col, text in items:
        pygame.draw.rect(surface, col, (x0, y0 + 3, 11, 11))
        lbl = small_font.render(text, True, (190, 190, 190))
        surface.blit(lbl, (x0 + 16, y0))
        y0 += 19

    eq1 = small_font.render(
        "Binet equation:  d\u00b2u/d\u03c6\u00b2 = -u + (3/2)\u00b7rs\u00b7u\u00b2    [u = 1/r,  rs = 2GM/c\u00b2]",
        True, (140, 140, 150))
    eq2 = small_font.render(
        "Integrated with 4th-order Runge-Kutta  |  impact parameter b = L/E",
        True, (120, 120, 130))
    surface.blit(eq1, (12, HEIGHT - 38))
    surface.blit(eq2, (12, HEIGHT - 20))

def main():
    pygame.init()
    screen = pygame.display.set_mode((WIDTH, HEIGHT))
    pygame.display.set_caption("2D Black Hole — Gravitational Lensing (Schwarzschild GR)")
    clock = pygame.time.Clock()

    font       = pygame.font.SysFont("monospace", 15, bold=True)
    small_font = pygame.font.SysFont("monospace", 12)

    print("Tracing photon geodesics…")
    rays = generate_rays(n_rays=90)
    captured_count = sum(1 for _, cap, _ in rays if cap)
    print(f"  Done — {len(rays)} rays  |  {captured_count} captured  |  "
          f"{len(rays)-captured_count} escaped")

    random.seed(7)
    stars = [(random.randint(0, WIDTH-1),
              random.randint(0, HEIGHT-1),
              random.randint(90, 240)) for _ in range(280)]

    scene = pygame.Surface((WIDTH, HEIGHT))
    scene.fill(BG_COLOR)
    draw_stars(scene, stars)
    draw_accretion_disk(scene)
    draw_circle_ring(scene, (255, 160,  30), R_ISCO,    width=1)
    draw_circle_ring(scene, (200, 200, 255), R_PHOTON,  width=1)
    draw_filled_circle(scene, (0, 0, 0), RS)
    draw_circle_ring(scene,   (160, 180, 255), RS, width=2)
    bh_sx, bh_sy = world_to_screen(0, 0)
    pygame.draw.circle(scene, (255, 255, 255), (bh_sx, bh_sy), 2)

    title = font.render(
        "Schwarzschild Black Hole  —  2D Gravitational Lensing",
        True, (210, 220, 255))
    scene.blit(title, (WIDTH//2 - title.get_width()//2, 6))
    draw_legend(scene, small_font)

    
    ray_points = [pts for pts, _, _ in rays]
    ray_meta   = [(cap, b) for _, cap, b in rays]
    positions  = [ -random.randint(0, 80) for _ in rays ]
    # speed per ray (larger impact param -> faster apparent motion)
    speeds = []
    for _, b in ray_meta:
        # normalise b for speed mapping
        bnorm = (b - (1.0 * RS)) / (5.0 * RS)
        bnorm = max(0.0, min(1.0, bnorm))
        speeds.append(0.5 + 4.0 * bnorm)

    trail_len = 36

    print("Window open. Press ESC or close to quit.")
    running = True
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE:
                running = False

        screen.blit(scene, (0, 0))

        # Animated Rays 
        for idx, pts in enumerate(ray_points):
            cap, b = ray_meta[idx]
            pos = positions[idx]
            pos += speeds[idx]
            positions[idx] = pos

            if pos < 1:
                continue

            head = int(pos)
            if head >= len(pts):
                # reset after the ray finishes its path
                positions[idx] = -random.randint(8, 60)
                continue

            
            trail_surf = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
            start = max(0, head - trail_len)
            for i in range(start, head - 1):
                p0 = pts[i]
                p1 = pts[i+1]
                a = int(255 * (i - start) / max(1, (head - start)))
                col = ray_color(b, cap)
                col_alpha = (col[0], col[1], col[2], min(230, a + 30))
                pygame.draw.line(trail_surf, col_alpha,
                                 world_to_screen(p0[0], p0[1]),
                                 world_to_screen(p1[0], p1[1]), 2)

            
            screen.blit(trail_surf, (0, 0))
            hx, hy = pts[head]
            pygame.draw.circle(screen, (255, 255, 255), world_to_screen(hx, hy), 2)

        pygame.display.flip()
        clock.tick(60)

    pygame.quit()
    sys.exit(0)

if __name__ == "__main__":
    main()
