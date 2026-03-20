<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const isMenuOpen = ref(false)
const heroCanvas = ref(null)

let heroAnimationFrame = 0
let heroResizeFrame = 0
let stopHeroNetwork = null

const closeMenu = () => {
  isMenuOpen.value = false
}

const navItems = [
  { label: 'Home', href: '#home' },
  { label: 'Sobre m\u00ed', href: '#sobre-mi' },
  { label: 'Proyectos', href: '#proyectos' },
  { label: 'Caso de estudio', href: '#caso-estudio' },
  { label: 'Skills', href: '#skills' },
  { label: 'Contacto', href: '#contacto' },
]

const projects = [
  {
    name: 'GuauRder\u00eda',
    summary:
      'Plataforma web para gesti\u00f3n de hotel y guarder\u00eda canina, dise\u00f1ada para convertir visitas en contactos con una experiencia clara, confiable y orientada a resultados.',
    role: 'Product Owner',
    responsibilities: [
      'Definici\u00f3n, priorizaci\u00f3n y refinamiento del backlog de producto.',
      'Redacci\u00f3n de historias de usuario y criterios de aceptaci\u00f3n claros.',
      'Planificaci\u00f3n de roadmap y organizaci\u00f3n de sprints.',
      'Priorizaci\u00f3n de funcionalidades con MoSCoW.',
      'Definici\u00f3n de estrategia de producto orientada a la conversi\u00f3n de leads.',
    ],
    outcome:
      'Se defini\u00f3 un MVP funcional enfocado en la captaci\u00f3n de clientes, optimizando el recorrido del usuario desde el descubrimiento hasta el contacto y reduciendo fricci\u00f3n.',
    reports: [
      { label: 'Informe funcional', href: '/reports/guaurderia1.pdf' },
      { label: 'Informe de producto', href: '/reports/guaurderia2.pdf' },
    ],
  },
  {
    name: 'Evolutech',
    summary:
      'Plataforma digital orientada a la gesti\u00f3n inteligente de escritorios conectados mediante IoT y anal\u00edtica de datos.',
    role: 'Product Owner',
    responsibilities: [
      'Definici\u00f3n de visi\u00f3n de producto alineada a objetivos de negocio.',
      'Identificaci\u00f3n de oportunidades estrat\u00e9gicas basadas en datos.',
      'Dise\u00f1o de propuesta de valor centrada en eficiencia operativa y experiencia del usuario.',
      'Enfoque en toma de decisiones data-driven.',
    ],
    outcome:
      'Se dise\u00f1\u00f3 una soluci\u00f3n orientada a optimizar procesos, reducir costos de mantenimiento y generar valor mediante an\u00e1lisis predictivo.',
    reports: [
      { label: 'Informe del proyecto', href: '/reports/evolutech.pdf' },
    ],
  },
  {
    name: 'Plataforma EdTech',
    summary:
      'Plataforma educativa digital enfocada en la personalizaci\u00f3n del aprendizaje para estudiantes de nivel secundario.',
    role: 'Product Owner',
    responsibilities: [
      'Definici\u00f3n y priorizaci\u00f3n del MVP.',
      'Identificaci\u00f3n y alineaci\u00f3n de stakeholders clave.',
      'Construcci\u00f3n de roadmap de producto.',
      'Aplicaci\u00f3n de User Story Mapping e Impact Mapping.',
    ],
    outcome:
      'Se logr\u00f3 alinear a los equipos en torno a una visi\u00f3n compartida del producto, definiendo un MVP orientado a validar la propuesta de valor.',
    reports: [
      { label: 'Informe del proyecto', href: '/reports/edtech.pdf' },
    ],
  },
]

const skillGroups = [
  {
    title: 'Gesti\u00f3n de Producto',
    items: [
      'Product Backlog Management',
      'Priorizaci\u00f3n basada en valor (MoSCoW)',
      'Definici\u00f3n de MVP',
      'Roadmapping de producto',
      'Levantamiento de requerimientos',
      'Gesti\u00f3n de stakeholders',
    ],
  },
  {
    title: 'Metodolog\u00edas y Frameworks',
    items: ['Scrum', 'Kanban', 'Agile Inception', 'Lean Startup', 'Design Thinking'],
  },
  {
    title: 'Investigaci\u00f3n y Usuario',
    items: [
      'User Story Mapping',
      'Empathy Mapping',
      'Definici\u00f3n de hip\u00f3tesis',
      'Validaci\u00f3n de soluciones',
      'Enfoque centrado en el usuario',
    ],
  },
  {
    title: 'M\u00e9tricas y An\u00e1lisis',
    items: [
      'Definici\u00f3n de KPIs',
      'M\u00e9tricas de conversi\u00f3n',
      'An\u00e1lisis de comportamiento de usuario',
      'Toma de decisiones basada en datos',
    ],
  },
  {
    title: 'Herramientas',
    items: ['Jira', 'Miro', 'Google Forms'],
  },
]

const setupHeroNetwork = () => {
  const canvas = heroCanvas.value
  if (!canvas) return () => {}

  const context = canvas.getContext('2d')
  const host = canvas.parentElement
  if (!context || !host) return () => {}

  const mobileQuery = window.matchMedia('(max-width: 640px)')
  const reduceMotionQuery = window.matchMedia('(prefers-reduced-motion: reduce)')

  let width = 0
  let height = 0
  let particles = []
  let reducedMotion = reduceMotionQuery.matches
  let lastTime = performance.now()
  let pointer = { x: 0, y: 0, active: false }

  const createParticle = () => {
    const mobile = mobileQuery.matches
    const velocity = mobile ? 0.058 : 0.084
    const safeZone = !mobile && Math.random() < 0.38
    const startX = safeZone ? width * (0.44 + Math.random() * 0.5) : Math.random() * width
    const startY = Math.random() * height

    return {
      x: startX,
      y: startY,
      vx: (Math.random() - 0.5) * velocity,
      vy: (Math.random() - 0.5) * velocity,
      radius: mobile ? 1.9 + Math.random() * 1.3 : 2.1 + Math.random() * 1.8,
      glow: 4 + Math.random() * 6,
      pulseOffset: Math.random() * Math.PI * 2,
    }
  }
  const syncPointer = (clientX, clientY) => {
    const bounds = canvas.getBoundingClientRect()
    pointer.x = clientX - bounds.left
    pointer.y = clientY - bounds.top
    pointer.active = true
  }

  const handlePointerMove = (event) => {
    syncPointer(event.clientX, event.clientY)
  }

  const handlePointerLeave = () => {
    pointer.active = false
  }

  const handleTouchMove = (event) => {
    if (!event.touches.length) return
    const touch = event.touches[0]
    syncPointer(touch.clientX, touch.clientY)
  }

  const handleTouchEnd = () => {
    pointer.active = false
  }

  const resize = () => {
    const ratio = Math.min(window.devicePixelRatio || 1, 2)
    const bounds = canvas.getBoundingClientRect()

    width = Math.max(bounds.width, 1)
    height = Math.max(bounds.height, 1)

    canvas.width = Math.round(width * ratio)
    canvas.height = Math.round(height * ratio)
    context.setTransform(ratio, 0, 0, ratio, 0, 0)

    const count = mobileQuery.matches ? 38 : 85
    particles = Array.from({ length: count }, createParticle)
  }

  const scheduleResize = () => {
    cancelAnimationFrame(heroResizeFrame)
    heroResizeFrame = requestAnimationFrame(resize)
  }

  const draw = (time) => {
    if (reducedMotion) return

    const delta = Math.min(time - lastTime, 32)
    lastTime = time

    context.clearRect(0, 0, width, height)

    const linkDistance = mobileQuery.matches ? 142 : 182
    const interactionRadius = mobileQuery.matches ? 158 : 194
    const interactionShift = mobileQuery.matches ? 18 : 24

    for (const particle of particles) {
      particle.x += particle.vx * delta
      particle.y += particle.vy * delta

      if (particle.x < -24 || particle.x > width + 24) particle.vx *= -1
      if (particle.y < -24 || particle.y > height + 24) particle.vy *= -1

      particle.x = Math.max(-24, Math.min(width + 24, particle.x))
      particle.y = Math.max(-24, Math.min(height + 24, particle.y))
    }

    const displayParticles = particles.map((particle) => {
      let renderX = particle.x
      let renderY = particle.y

      if (pointer.active) {
        const dx = renderX - pointer.x
        const dy = renderY - pointer.y
        const distance = Math.hypot(dx, dy)

        if (distance < interactionRadius && distance > 0) {
          const force = Math.pow(1 - distance / interactionRadius, 1.35)
          const shift = force * interactionShift
          renderX += (dx / distance) * shift
          renderY += (dy / distance) * shift
        }
      }

      return {
        particle,
        x: renderX,
        y: renderY,
      }
    })

    for (let index = 0; index < displayParticles.length; index += 1) {
      const source = displayParticles[index]

      for (let next = index + 1; next < displayParticles.length; next += 1) {
        const target = displayParticles[next]
        const dx = target.x - source.x
        const dy = target.y - source.y
        const distance = Math.hypot(dx, dy)

        if (distance > linkDistance) continue

        const centerX = (source.x + target.x) / 2
        const centerY = (source.y + target.y) / 2
        const textQuietZone = !mobileQuery.matches && centerX < width * 0.42 && centerY > height * 0.16 && centerY < height * 0.82
        const alpha = (1 - distance / linkDistance) * (textQuietZone ? 0.14 : 0.24)

        context.beginPath()
        context.moveTo(source.x, source.y)
        context.lineTo(target.x, target.y)
        context.strokeStyle = `rgba(110, 103, 94, ${alpha.toFixed(3)})`
        context.lineWidth = 1
        context.stroke()
      }
    }

    if (pointer.active) {
      for (const particle of displayParticles) {
        const dx = particle.x - pointer.x
        const dy = particle.y - pointer.y
        const distance = Math.hypot(dx, dy)

        if (distance > interactionRadius * 0.72) continue

        const alpha = (1 - distance / (interactionRadius * 0.72)) * 0.16
        context.beginPath()
        context.moveTo(pointer.x, pointer.y)
        context.lineTo(particle.x, particle.y)
        context.strokeStyle = `rgba(31, 59, 115, ${alpha.toFixed(3)})`
        context.lineWidth = 1
        context.stroke()
      }

      context.beginPath()
      context.arc(pointer.x, pointer.y, mobileQuery.matches ? 34 : 42, 0, Math.PI * 2)
      context.fillStyle = 'rgba(31, 59, 115, 0.04)'
      context.fill()
    }

    for (const entry of displayParticles) {
      const particle = entry.particle
      const pulse = 0.85 + Math.sin(time * 0.0032 + particle.pulseOffset) * 0.2
      const textQuietZone = !mobileQuery.matches && entry.x < width * 0.4 && entry.y > height * 0.14 && entry.y < height * 0.84
      const fade = (mobileQuery.matches ? 0.84 : 1) * (textQuietZone ? 0.62 : 1)
      const radius = particle.radius * pulse
      const glowRadius = particle.glow + Math.sin(time * 0.0024 + particle.pulseOffset) * 1.8
      const pointAlpha = 0.58 * fade
      const glowAlpha = 0.11 * fade

      context.beginPath()
      context.arc(entry.x, entry.y, glowRadius, 0, Math.PI * 2)
      context.fillStyle = `rgba(31, 59, 115, ${glowAlpha.toFixed(3)})`
      context.fill()

      context.beginPath()
      context.arc(entry.x, entry.y, radius, 0, Math.PI * 2)
      context.fillStyle = `rgba(31, 59, 115, ${pointAlpha.toFixed(3)})`
      context.fill()
    }

    heroAnimationFrame = requestAnimationFrame(draw)
  }
  const handleReducedMotion = (event) => {
    reducedMotion = event.matches

    if (reducedMotion) {
      cancelAnimationFrame(heroAnimationFrame)
      context.clearRect(0, 0, width, height)
      return
    }

    lastTime = performance.now()
    heroAnimationFrame = requestAnimationFrame(draw)
  }

  resize()

  if (!reducedMotion) {
    heroAnimationFrame = requestAnimationFrame(draw)
  }

  host.addEventListener('pointermove', handlePointerMove)
  host.addEventListener('pointerenter', handlePointerMove)
  host.addEventListener('pointerleave', handlePointerLeave)
  host.addEventListener('touchstart', handleTouchMove, { passive: true })
  host.addEventListener('touchmove', handleTouchMove, { passive: true })
  host.addEventListener('touchend', handleTouchEnd, { passive: true })
  window.addEventListener('resize', scheduleResize)
  mobileQuery.addEventListener('change', scheduleResize)
  reduceMotionQuery.addEventListener('change', handleReducedMotion)

  return () => {
    cancelAnimationFrame(heroAnimationFrame)
    cancelAnimationFrame(heroResizeFrame)
    host.removeEventListener('pointermove', handlePointerMove)
    host.removeEventListener('pointerenter', handlePointerMove)
    host.removeEventListener('pointerleave', handlePointerLeave)
    host.removeEventListener('touchstart', handleTouchMove)
    host.removeEventListener('touchmove', handleTouchMove)
    host.removeEventListener('touchend', handleTouchEnd)
    window.removeEventListener('resize', scheduleResize)
    mobileQuery.removeEventListener('change', scheduleResize)
    reduceMotionQuery.removeEventListener('change', handleReducedMotion)
  }
}

onMounted(() => {
  stopHeroNetwork = setupHeroNetwork()
})

onBeforeUnmount(() => {
  if (typeof stopHeroNetwork === 'function') {
    stopHeroNetwork()
  }
})
</script>

<template>
  <div class="page-shell">
    <header class="topbar">
      <div class="container nav-wrap">
        <a class="brand" href="#home" @click="closeMenu">
          <span class="brand-name">Benjam&iacute;n Ruiz</span>
          <span class="brand-role">Product Owner Junior</span>
        </a>

        <button
          class="menu-toggle"
          type="button"
          :class="{ 'is-open': isMenuOpen }"
          :aria-expanded="isMenuOpen"
          aria-label="Abrir men&uacute;"
          @click="isMenuOpen = !isMenuOpen"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>

        <nav class="nav desktop-nav">
          <a v-for="item in navItems" :key="item.href" :href="item.href">{{ item.label }}</a>
        </nav>
      </div>
    </header>

    <transition name="menu-fade">
      <div v-if="isMenuOpen" class="mobile-menu-shell" @click.self="closeMenu">
        <div class="mobile-menu-panel">
          <a
            v-for="item in navItems"
            :key="item.href"
            :href="item.href"
            class="mobile-menu-link"
            @click="closeMenu"
          >
            {{ item.label }}
          </a>
        </div>
      </div>
    </transition>

    <main class="container main-flow">
      <section id="home" class="hero">
        <canvas ref="heroCanvas" class="hero-network" aria-hidden="true"></canvas>
        <div class="hero-inner">
          <div class="hero-copy">
            <p class="kicker">Product Owner Junior</p>
            <h1>
              Dise&ntilde;o y priorizaci&oacute;n de
              <span>productos digitales</span>
              con foco en valor.
            </h1>
            <p class="lead">
              Product Owner en formaci&oacute;n con enfoque en entrega de valor, priorizaci&oacute;n estrat&eacute;gica,
              investigaci&oacute;n y validaci&oacute;n de soluciones centradas en el usuario.
            </p>
            <div class="hero-actions">
              <a class="button primary" href="#proyectos">Ver proyectos</a>
              <a class="button secondary" href="/CVPO.pdf" target="_blank" rel="noreferrer">
                Descargar CV
              </a>
            </div>
          </div>

          <aside class="hero-panel">
            <div class="hero-panel-card">
              <p class="panel-label">Perfil</p>
              <p class="panel-text">
                Conecto negocio, usuario y ejecuci&oacute;n para aterrizar decisiones de producto con
                criterio, claridad y sentido de prioridad.
              </p>
              <div class="panel-metrics">
                <div>
                  <strong>Backlog</strong>
                  <span>Priorizaci&oacute;n y refinamiento</span>
                </div>
                <div>
                  <strong>Investigaci&oacute;n</strong>
                  <span>Hip&oacute;tesis, validaci&oacute;n y aprendizaje</span>
                </div>
              </div>
            </div>
          </aside>
        </div>
      </section>

      <section id="sobre-mi" class="section">
        <div class="section-heading">
          <p>Sobre m&iacute;</p>
          <h2>Una mirada estrat&eacute;gica del producto con ejecuci&oacute;n concreta.</h2>
        </div>
        <div class="about-grid">
          <article class="surface-card prose-card">
            <p>
              Soy Product Owner en formaci&oacute;n con experiencia en el desarrollo de productos
              digitales utilizando metodolog&iacute;as &aacute;giles y enfoques centrados en el usuario.
            </p>
            <p>
              Durante mi formaci&oacute;n, he participado en definici&oacute;n de visi&oacute;n de producto,
              priorizaci&oacute;n de backlog, construcci&oacute;n de MVPs y validaci&oacute;n de soluciones mediante
              m&eacute;tricas, comprendiendo el ciclo completo de desarrollo de productos digitales.
            </p>
          </article>
          <article class="surface-card prose-card">
            <p>
              Me enfoco en identificar problemas relevantes, alinear objetivos de negocio y
              colaborar con equipos multidisciplinarios para entregar valor de forma incremental.
            </p>
            <p>
              Mi objetivo es desarrollarme en entornos din&aacute;micos como Mercado Libre, aportando
              desde una mirada estrat&eacute;gica del producto y contribuyendo a la mejora continua de
              la experiencia del usuario.
            </p>
          </article>
        </div>
      </section>

      <section id="proyectos" class="section">
        <div class="section-heading">
          <p>Proyectos</p>
          <h2>Experiencias de producto construidas con foco en negocio, usuario y resultado.</h2>
        </div>
        <div class="project-list">
          <article v-for="project in projects" :key="project.name" class="surface-card project-card">
            <div class="project-top">
              <div>
                <h3>{{ project.name }}</h3>
                <p class="project-summary">{{ project.summary }}</p>
              </div>

              <div class="project-side">
                <span class="role-chip">{{ project.role }}</span>

                <details v-if="project.reports?.length" class="resource-menu">
                  <summary class="resource-trigger">
                    Recursos
                    <span>{{ project.reports.length }}</span>
                  </summary>
                  <div class="resource-menu-panel">
                    <a
                      v-for="report in project.reports"
                      :key="report.href"
                      :href="report.href"
                      class="resource-item"
                      target="_blank"
                      rel="noreferrer"
                    >
                      {{ report.label }}
                    </a>
                  </div>
                </details>
              </div>
            </div>

            <div class="project-content">
              <div>
                <h4>Responsabilidades</h4>
                <ul>
                  <li v-for="item in project.responsibilities" :key="item">{{ item }}</li>
                </ul>
              </div>
              <div>
                <h4>Resultado</h4>
                <p>{{ project.outcome }}</p>
              </div>
            </div>
          </article>
        </div>
      </section>

      <section id="caso-estudio" class="section">
        <div class="section-heading">
          <p>Caso de estudio</p>
          <h2>GuauRder&iacute;a: priorizaci&oacute;n del MVP para reducir fricci&oacute;n y mejorar conversi&oacute;n.</h2>
        </div>
        <article class="surface-card case-study">
          <div class="case-grid">
            <div>
              <h3>Problema</h3>
              <p>
                Los usuarios no contaban con suficiente informaci&oacute;n ni confianza para contratar
                servicios de guarder&iacute;a canina, generando fricci&oacute;n en el proceso de decisi&oacute;n.
              </p>
            </div>
            <div>
              <h3>Soluci&oacute;n</h3>
              <p>
                Se dise&ntilde;&oacute; una plataforma web orientada a la conversi&oacute;n, priorizando
                funcionalidades clave como informaci&oacute;n del servicio, requisitos de admisi&oacute;n y
                canales de contacto directo.
              </p>
            </div>
            <div>
              <h3>Decisi&oacute;n clave</h3>
              <p>
                Se prioriz&oacute; un MVP sin autenticaci&oacute;n, eliminando barreras de entrada y reduciendo
                la fricci&oacute;n en el proceso de captaci&oacute;n de clientes.
              </p>
            </div>
            <div>
              <h3>M&eacute;tricas</h3>
              <ul>
                <li>Tasa de conversi&oacute;n de visitantes a contactos.</li>
                <li>Interacci&oacute;n con canales de contacto como WhatsApp y formularios.</li>
              </ul>
            </div>
          </div>
          <div class="case-learnings">
            <h3>Aprendizajes</h3>
            <ul>
              <li>Importancia de priorizar valor de negocio por sobre complejidad t&eacute;cnica.</li>
              <li>La validaci&oacute;n temprana reduce incertidumbre y mejora decisiones de producto.</li>
              <li>Usuario, negocio y tecnolog&iacute;a deben alinearse en cada decisi&oacute;n.</li>
            </ul>
          </div>
        </article>
      </section>

      <section id="skills" class="section">
        <div class="section-heading">
          <p>Skills</p>
          <h2>Capacidades clave para investigaci&oacute;n, ejecuci&oacute;n y gesti&oacute;n de producto.</h2>
        </div>
        <div class="skills-grid">
          <article v-for="group in skillGroups" :key="group.title" class="surface-card skill-card">
            <h3>{{ group.title }}</h3>
            <ul>
              <li v-for="item in group.items" :key="item">{{ item }}</li>
            </ul>
          </article>
        </div>
      </section>

      <section id="contacto" class="section">
        <div class="section-heading">
          <p>Contacto</p>
          <h2>Disponible para oportunidades en producto digital y colaboraci&oacute;n.</h2>
        </div>
        <article class="surface-card contact-card">
          <div class="contact-layout">
            <div class="contact-copy">
              <p>
                Estoy disponible para oportunidades como Product Owner Junior, colaboraciones en
                proyectos digitales o roles relacionados con gesti&oacute;n de producto.
              </p>
              <div class="contact-note">
                <strong>Disponibilidad</strong>
                <span>Abierto a conversaciones sobre producto, validaci&oacute;n y construcci&oacute;n de MVPs.</span>
              </div>
              <a class="button primary" href="/CVPO.pdf" target="_blank" rel="noreferrer">Ver CV</a>
            </div>

            <div class="contact-grid">
              <a class="contact-item" href="mailto:benjamin.ruizgarrido@gmail.com">
                <span class="contact-icon" aria-hidden="true">
                  <img src="/icons/gmail.png" alt="" />
                </span>
                <span class="contact-meta">
                  <strong>Email</strong>
                  <span>benjamin.ruizgarrido@gmail.com</span>
                </span>
              </a>

              <a class="contact-item" href="https://wa.me/56954759527" target="_blank" rel="noreferrer">
                <span class="contact-icon" aria-hidden="true">
                  <img src="/icons/whatsapp.png" alt="" />
                </span>
                <span class="contact-meta">
                  <strong>WhatsApp</strong>
                  <span>+56 9 5475 9527</span>
                </span>
              </a>
            </div>
          </div>
        </article>
      </section>
    </main>
  </div>
</template>


