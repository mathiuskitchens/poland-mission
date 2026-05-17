<script>
  import { onMount, onDestroy, createEventDispatcher } from 'svelte'
  import * as THREE from 'three'

  const dispatch = createEventDispatcher()

  const SEGMENTS = [
    { label: '$25',         amount: 25,   color: '#4fc3f7', dark: false },
    { label: '$50',         amount: 50,   color: '#66bb6a', dark: false },
    { label: '$100',        amount: 100,  color: '#ffa726', dark: false },
    { label: '$150',        amount: 150,  color: '#ab47bc', dark: false },
    { label: '$200',        amount: 200,  color: '#ef5350', dark: false },
    { label: '$500',        amount: 500,  color: '#d4a843', dark: false },
    { label: '$75',         amount: 75,   color: '#26c6da', dark: false },
    { label: 'Pray for Us', amount: 0,    color: '#eceff1', dark: true  },
  ]
  const N = SEGMENTS.length
  const SEG_ANGLE = (Math.PI * 2) / N

  let container
  let renderer, scene, camera, animId
  let wheelMesh
  let isSpinning = false
  let result = null
  let currentRotation = 0

  onMount(() => {
    init()
    window.addEventListener('resize', onResize)
    return () => {
      window.removeEventListener('resize', onResize)
      cancelAnimationFrame(animId)
      renderer.dispose()
    }
  })

  function buildWheelTexture() {
    const SIZE = 1024
    const c = document.createElement('canvas')
    c.width = SIZE
    c.height = SIZE
    const ctx = c.getContext('2d')
    const cx = SIZE / 2, cy = SIZE / 2, r = SIZE / 2 - 8

    SEGMENTS.forEach((seg, i) => {
      const a0 = i * SEG_ANGLE
      const a1 = a0 + SEG_ANGLE

      // Segment fill
      ctx.beginPath()
      ctx.moveTo(cx, cy)
      ctx.arc(cx, cy, r, a0, a1)
      ctx.closePath()
      ctx.fillStyle = seg.color
      ctx.fill()

      // Divider lines
      ctx.strokeStyle = 'rgba(255,255,255,0.9)'
      ctx.lineWidth = 3
      ctx.stroke()

      // Label text
      const mid = a0 + SEG_ANGLE / 2
      const tr = r * 0.62
      const tx = cx + tr * Math.cos(mid)
      const ty = cy + tr * Math.sin(mid)

      ctx.save()
      ctx.translate(tx, ty)
      ctx.rotate(mid + Math.PI / 2)
      ctx.fillStyle = seg.dark ? '#1a1a2e' : '#0d1333'
      ctx.font = `bold ${seg.label.length > 5 ? 36 : 54}px Inter, Arial`
      ctx.textAlign = 'center'
      ctx.textBaseline = 'middle'
      ctx.fillText(seg.label, 0, 0)
      ctx.restore()
    })

    // Center hub
    const hubR = 48
    ctx.beginPath()
    ctx.arc(cx, cy, hubR, 0, Math.PI * 2)
    const hubGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, hubR)
    hubGrad.addColorStop(0, '#f0c868')
    hubGrad.addColorStop(1, '#a07820')
    ctx.fillStyle = hubGrad
    ctx.fill()
    ctx.strokeStyle = '#ffffff'
    ctx.lineWidth = 3
    ctx.stroke()

    return c
  }

  function init() {
    const w = container.clientWidth
    const h = container.clientHeight

    scene = new THREE.Scene()
    scene.background = new THREE.Color(0x08102a)

    camera = new THREE.PerspectiveCamera(52, w / h, 0.1, 100)
    camera.position.set(0, 0.3, 5.8)
    camera.lookAt(0, 0, 0)

    renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setSize(w, h)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    container.appendChild(renderer.domElement)

    // Wheel face (CircleGeometry)
    const tex = new THREE.CanvasTexture(buildWheelTexture())
    const wheelGeo = new THREE.CircleGeometry(2.5, 128)
    const wheelMat = new THREE.MeshBasicMaterial({ map: tex, side: THREE.DoubleSide })
    wheelMesh = new THREE.Mesh(wheelGeo, wheelMat)
    scene.add(wheelMesh)

    // Gold rim
    const rimGeo = new THREE.TorusGeometry(2.5, 0.12, 8, 96)
    const rimMat = new THREE.MeshStandardMaterial({ color: 0xd4a843, metalness: 0.85, roughness: 0.15 })
    scene.add(new THREE.Mesh(rimGeo, rimMat))

    // Outer decorative ring
    const outerGeo = new THREE.TorusGeometry(2.62, 0.04, 6, 96)
    const outerMat = new THREE.MeshStandardMaterial({ color: 0xf0c868, metalness: 0.9, roughness: 0.1 })
    scene.add(new THREE.Mesh(outerGeo, outerMat))

    // Pointer arrow (right side) — large triangular pointer
    const pointerShape = new THREE.Shape()
    pointerShape.moveTo(0, 0)
    pointerShape.lineTo(0.7, 0.3)
    pointerShape.lineTo(0.7, -0.3)
    pointerShape.closePath()
    const arrowGeo = new THREE.ExtrudeGeometry(pointerShape, { depth: 0.12, bevelEnabled: true, bevelThickness: 0.03, bevelSize: 0.03, bevelSegments: 3 })
    const arrowMat = new THREE.MeshStandardMaterial({ color: 0xdc143c, metalness: 0.5, roughness: 0.3 })
    const arrow = new THREE.Mesh(arrowGeo, arrowMat)
    arrow.position.set(2.75, 0, 0.1)
    scene.add(arrow)

    // Arrow outline/border for contrast
    const outerPointerShape = new THREE.Shape()
    outerPointerShape.moveTo(-0.06, 0)
    outerPointerShape.lineTo(0.76, 0.36)
    outerPointerShape.lineTo(0.76, -0.36)
    outerPointerShape.closePath()
    const borderGeo = new THREE.ExtrudeGeometry(outerPointerShape, { depth: 0.06, bevelEnabled: false })
    const borderMat = new THREE.MeshStandardMaterial({ color: 0xffffff, metalness: 0.3, roughness: 0.5 })
    const border = new THREE.Mesh(borderGeo, borderMat)
    border.position.set(2.75, 0, 0.05)
    scene.add(border)

    // Lights
    scene.add(new THREE.AmbientLight(0xffffff, 1.4))
    const key = new THREE.DirectionalLight(0xfff5dd, 1.2)
    key.position.set(3, 4, 6)
    scene.add(key)
    const fill = new THREE.PointLight(0x334488, 0.8, 20)
    fill.position.set(-4, -2, 2)
    scene.add(fill)

    // Background particles
    const pGeo = new THREE.BufferGeometry()
    const pPos = []
    for (let i = 0; i < 800; i++) {
      pPos.push((Math.random() - 0.5) * 40, (Math.random() - 0.5) * 40, (Math.random() - 0.5) * 20 - 5)
    }
    pGeo.setAttribute('position', new THREE.Float32BufferAttribute(pPos, 3))
    const pMat = new THREE.PointsMaterial({ color: 0x6699cc, size: 0.06, transparent: true, opacity: 0.5 })
    scene.add(new THREE.Points(pGeo, pMat))

    // Slight idle tilt
    wheelMesh.rotation.x = 0.12

    renderLoop()
  }

  function renderLoop() {
    animId = requestAnimationFrame(renderLoop)
    renderer.render(scene, camera)
  }

  function spin() {
    if (isSpinning) return
    isSpinning = true
    result = null

    const totalSpins = 5 + Math.random() * 5
    const randomAngle = Math.random() * Math.PI * 2
    const targetRotation = currentRotation + totalSpins * Math.PI * 2 + randomAngle
    const startRotation = currentRotation
    const startTime = performance.now()
    const duration = 4500 + Math.random() * 1000

    function spinFrame(now) {
      const elapsed = now - startTime
      const t = Math.min(elapsed / duration, 1)
      // Ease out quint
      const eased = 1 - Math.pow(1 - t, 5)
      currentRotation = startRotation + (targetRotation - startRotation) * eased
      wheelMesh.rotation.z = currentRotation

      if (t < 1) {
        requestAnimationFrame(spinFrame)
      } else {
        currentRotation = targetRotation
        wheelMesh.rotation.z = currentRotation
        isSpinning = false
        resolveResult()
      }
    }

    requestAnimationFrame(spinFrame)
  }

  function resolveResult() {
    // After rotation φ CCW, the canvas CW angle at the pointer (right side) is φ mod 2π
    const phi = ((currentRotation % (Math.PI * 2)) + Math.PI * 2) % (Math.PI * 2)
    const idx = Math.floor(phi / SEG_ANGLE) % N
    result = SEGMENTS[idx]
  }

  function onResize() {
    if (!container || !renderer) return
    const w = container.clientWidth
    const h = container.clientHeight
    camera.aspect = w / h
    camera.updateProjectionMatrix()
    renderer.setSize(w, h)
  }
</script>

<div class="wheel-section">
  <div class="canvas-wrap" bind:this={container}></div>

  <div class="controls">
    {#if result === null}
      <button class="spin-btn" on:click={spin} disabled={isSpinning}>
        {isSpinning ? 'Spinning...' : 'SPIN THE WHEEL'}
      </button>
    {:else}
      <div class="result-box" class:free={result.amount === 0}>
        {#if result.amount > 0}
          <p class="result-label">You landed on</p>
          <p class="result-amount">{result.label}</p>
          <p class="result-sub">That's your number! Every dollar brings the mission closer.</p>
          <div class="result-actions">
            <button class="btn-donate" on:click={() => dispatch('donate')}>Donate {result.label} Now</button>
            <button class="spin-again" on:click={() => { result = null }}>Spin Again</button>
          </div>
        {:else}
          <p class="result-label">You landed on</p>
          <p class="result-amount">Prayer!</p>
          <p class="result-sub">Your prayers mean everything to our team. Would you also consider a small gift?</p>
          <div class="result-actions">
            <button class="btn-donate" on:click={() => dispatch('donate')}>Give Anyway</button>
            <button class="spin-again" on:click={() => { result = null }}>Try Again</button>
          </div>
        {/if}
      </div>
    {/if}
  </div>
</div>

<style>
  .wheel-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2rem;
  }

  .canvas-wrap {
    width: 100%;
    max-width: 540px;
    aspect-ratio: 1.1 / 1;
    border-radius: 16px;
    overflow: hidden;
    box-shadow: 0 0 60px rgba(212, 168, 67, 0.15), 0 0 120px rgba(30, 80, 160, 0.2);
  }

  .canvas-wrap :global(canvas) {
    display: block;
    width: 100% !important;
    height: 100% !important;
  }

  .controls {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 160px;
    justify-content: center;
    width: 100%;
    max-width: 540px;
  }

  .spin-btn {
    padding: 1rem 3rem;
    font-size: 1.15rem;
    font-weight: 700;
    letter-spacing: 0.12em;
    background: linear-gradient(135deg, #d4a843 0%, #f0c868 50%, #a07820 100%);
    color: #0d1333;
    border-radius: 50px;
    text-transform: uppercase;
    box-shadow: 0 4px 24px rgba(212, 168, 67, 0.5);
    transition: transform 0.15s, box-shadow 0.15s, opacity 0.15s;
  }

  .spin-btn:hover:not(:disabled) {
    transform: translateY(-2px) scale(1.04);
    box-shadow: 0 8px 32px rgba(212, 168, 67, 0.7);
  }

  .spin-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .result-box {
    background: rgba(13, 19, 51, 0.9);
    border: 1px solid rgba(212, 168, 67, 0.35);
    border-radius: 16px;
    padding: 1.75rem 2rem;
    text-align: center;
    width: 100%;
    animation: fadeUp 0.4s ease;
  }

  .result-box.free {
    border-color: rgba(255, 255, 255, 0.2);
  }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  .result-label {
    font-size: 0.85rem;
    color: var(--muted, #7a8db5);
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 0.25rem;
  }

  .result-amount {
    font-family: 'Cinzel', serif;
    font-size: 2.6rem;
    font-weight: 700;
    color: #d4a843;
    margin-bottom: 0.5rem;
    line-height: 1.1;
  }

  .result-sub {
    font-size: 0.9rem;
    color: #aab8d8;
    margin-bottom: 1.25rem;
  }

  .result-actions {
    display: flex;
    gap: 0.75rem;
    justify-content: center;
    flex-wrap: wrap;
  }

  .btn-donate {
    padding: 0.7rem 1.6rem;
    background: linear-gradient(135deg, #dc143c, #a00028);
    color: #fff;
    border-radius: 8px;
    font-weight: 600;
    font-size: 0.9rem;
    box-shadow: 0 4px 16px rgba(220, 20, 60, 0.4);
    transition: transform 0.15s, box-shadow 0.15s;
  }

  .btn-donate:hover {
    transform: translateY(-1px);
    box-shadow: 0 6px 20px rgba(220, 20, 60, 0.55);
  }

  .spin-again {
    padding: 0.7rem 1.4rem;
    background: transparent;
    color: #aab8d8;
    border: 1px solid rgba(170, 184, 216, 0.3);
    border-radius: 8px;
    font-size: 0.9rem;
    transition: color 0.15s, border-color 0.15s;
  }

  .spin-again:hover {
    color: #fff;
    border-color: rgba(255, 255, 255, 0.5);
  }
</style>
