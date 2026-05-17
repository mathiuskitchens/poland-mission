<script>
  import { onMount, createEventDispatcher } from 'svelte'
  import * as THREE from 'three'

  const dispatch = createEventDispatcher()

  let container
  let renderer, scene, camera, animId
  let globeGroup
  let pinGroup = null
  let pulseRings = []
  let pulseT = 0
  let isFlying = false
  let flyStart = null
  let flyDuration = 2200
  let pinDropped = false
  let pinScale = 0
  let autoRotate = true

  const flyStartQuat = new THREE.Quaternion()
  const flyEndQuat = new THREE.Quaternion()
  const tmpQuat = new THREE.Quaternion()

  const POLAND_LAT = 51.9
  const POLAND_LON = 19.5

  // ── Continent outlines (lon, lat) — wide enough that Poland is on land ────
  const CONTINENTS = [
    // North America
    { pts: [
      [-168,66],[-160,71],[-145,71],[-128,71],[-110,72],[-95,74],[-80,73],
      [-72,78],[-62,82],[-55,76],[-55,65],[-60,60],[-67,55],[-72,50],
      [-68,46],[-60,46],[-55,50],[-58,44],[-65,42],[-75,38],[-80,30],
      [-82,26],[-88,29],[-94,29],[-98,25],[-105,22],[-112,28],[-117,32],
      [-122,37],[-124,42],[-124,48],[-130,54],[-138,58],[-148,60],
      [-160,60],[-168,66]
    ], fill: 'land' },

    // South America
    { pts: [
      [-78,12],[-72,12],[-62,11],[-55,7],[-50,2],[-46,-2],[-38,-7],
      [-37,-13],[-40,-22],[-46,-25],[-50,-30],[-58,-35],[-65,-42],
      [-72,-52],[-74,-50],[-73,-42],[-72,-32],[-72,-22],[-75,-14],
      [-78,-6],[-80,0],[-78,6],[-78,12]
    ], fill: 'land' },

    // Greater Europe + Western Russia (one big landmass; Poland sits inside)
    { pts: [
      [-10,36],[-9,38],[-9,43],[-2,43],[-2,48],[-5,50],[-7,54],
      [-10,54],[-6,58],[2,58],[5,62],[8,65],[12,68],[18,70],[25,71],
      [32,69],[40,67],[50,67],[58,67],[64,62],[62,55],[55,50],
      [48,48],[42,45],[38,42],[33,42],[28,40],[26,40],[22,38],
      [18,40],[15,37],[10,42],[3,42],[-2,36],[-10,36]
    ], fill: 'land' },

    // Africa
    { pts: [
      [-17,15],[-17,22],[-13,28],[-8,32],[-2,35],[10,33],[20,32],
      [25,32],[32,32],[33,30],[36,28],[40,18],[44,12],[48,8],[51,4],
      [50,-3],[48,-10],[44,-15],[40,-18],[40,-25],[35,-30],[28,-34],
      [22,-34],[18,-32],[15,-22],[13,-15],[12,-8],[10,-2],[8,3],
      [5,5],[2,6],[-2,6],[-5,7],[-8,5],[-11,5],[-13,7],[-15,10],
      [-17,12],[-17,15]
    ], fill: 'land' },

    // Asia (broad)
    { pts: [
      [62,55],[65,62],[75,68],[90,72],[110,72],[130,70],[142,68],
      [148,60],[145,52],[140,46],[136,40],[130,32],[124,24],
      [120,18],[112,8],[105,4],[100,8],[96,16],[92,22],[88,25],
      [84,18],[80,12],[78,8],[76,10],[72,16],[70,22],[68,26],
      [65,30],[60,36],[55,38],[52,42],[52,46],[55,50],[58,52],[62,55]
    ], fill: 'land' },

    // Indian subcontinent peninsula
    { pts: [
      [68,24],[72,18],[75,12],[78,8],[80,8],[82,11],[85,15],
      [88,20],[88,22],[85,22],[82,20],[78,20],[75,22],[72,24],[68,24]
    ], fill: 'land' },

    // SE Asia / Indonesia (rough)
    { pts: [
      [95,8],[100,5],[105,2],[110,3],[115,5],[120,2],[125,-2],
      [120,-6],[110,-8],[100,-3],[97,2],[95,8]
    ], fill: 'land' },

    // Australia
    { pts: [
      [114,-22],[120,-18],[128,-14],[135,-13],[140,-15],[143,-15],
      [148,-20],[153,-26],[153,-32],[148,-37],[141,-38],[135,-34],
      [128,-32],[122,-32],[116,-27],[114,-22]
    ], fill: 'land' },

    // Japan
    { pts: [[131,32],[135,34],[139,36],[141,40],[142,43],[140,45],[137,42],[134,38],[131,32]], fill: 'land' },

    // Greenland
    { pts: [[-50,60],[-30,60],[-20,68],[-18,76],[-25,82],[-40,82],[-55,78],[-58,72],[-55,65],[-50,60]], fill: 'ice' },

    // Antarctica
    { pts: [[-180,-65],[-120,-70],[-60,-72],[0,-70],[60,-68],[120,-70],[180,-65],[180,-90],[-180,-90],[-180,-65]], fill: 'ice' },
  ]

  // ── Earth canvas texture ─────────────────────────────────────────────────
  function buildEarthTexture() {
    const W = 2048, H = 1024
    const cv = document.createElement('canvas')
    cv.width = W; cv.height = H
    const ctx = cv.getContext('2d')

    const toXY = (lon, lat) => [(lon + 180) / 360 * W, (90 - lat) / 180 * H]

    // Ocean: layered blue, deeper near equator
    const ocean = ctx.createLinearGradient(0, 0, 0, H)
    ocean.addColorStop(0,    '#16263d')
    ocean.addColorStop(0.18, '#143356')
    ocean.addColorStop(0.42, '#0f3d72')
    ocean.addColorStop(0.5,  '#10437d')
    ocean.addColorStop(0.58, '#0f3d72')
    ocean.addColorStop(0.82, '#143356')
    ocean.addColorStop(1,    '#16263d')
    ctx.fillStyle = ocean
    ctx.fillRect(0, 0, W, H)

    // Soft ocean cloud-like variation
    for (let i = 0; i < 500; i++) {
      const x = Math.random() * W
      const y = Math.random() * H
      const r = Math.random() * 90 + 40
      ctx.fillStyle = `rgba(30,80,140,${Math.random() * 0.05})`
      ctx.beginPath()
      ctx.arc(x, y, r, 0, Math.PI * 2)
      ctx.fill()
    }

    const fillContinent = (pts, fillType) => {
      ctx.beginPath()
      pts.forEach(([lon, lat], i) => {
        const [x, y] = toXY(lon, lat)
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
      })
      ctx.closePath()

      if (fillType === 'ice') {
        const ice = ctx.createLinearGradient(0, 0, 0, H)
        ice.addColorStop(0, '#eef4f8')
        ice.addColorStop(0.5, '#d4e2eb')
        ice.addColorStop(1, '#eef4f8')
        ctx.fillStyle = ice
        ctx.fill()
        ctx.strokeStyle = 'rgba(190,210,225,0.5)'
        ctx.lineWidth = 1
        ctx.stroke()
      } else {
        const land = ctx.createLinearGradient(0, 0, 0, H)
        land.addColorStop(0,    '#4d5e3a')   // tundra
        land.addColorStop(0.18, '#5b7240')   // boreal
        land.addColorStop(0.32, '#6f8a46')   // temperate
        land.addColorStop(0.42, '#7d934a')
        land.addColorStop(0.5,  '#8a984a')   // sahel
        land.addColorStop(0.58, '#7d934a')
        land.addColorStop(0.68, '#6f8a46')
        land.addColorStop(0.82, '#5b7240')
        land.addColorStop(1,    '#4d5e3a')
        ctx.fillStyle = land
        ctx.fill()
        ctx.strokeStyle = 'rgba(45,60,30,0.55)'
        ctx.lineWidth = 1.4
        ctx.stroke()
      }
    }

    CONTINENTS.forEach(c => fillContinent(c.pts, c.fill))

    // Desert overlays (Sahara/Arabia + Outback + Gobi)
    const deserts = [
      { pts: [[-15,32],[40,32],[50,22],[40,14],[-15,18]], color: 'rgba(205,170,95,0.4)' },
      { pts: [[115,-20],[145,-20],[145,-32],[115,-30]], color: 'rgba(200,160,90,0.3)' },
      { pts: [[80,48],[110,48],[110,38],[80,38]],       color: 'rgba(190,160,90,0.18)' },
    ]
    deserts.forEach(({ pts, color }) => {
      ctx.beginPath()
      pts.forEach(([lon, lat], i) => {
        const [x, y] = toXY(lon, lat)
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
      })
      ctx.closePath()
      ctx.fillStyle = color
      ctx.fill()
    })

    // Subtle terrain noise (light)
    ctx.globalCompositeOperation = 'overlay'
    for (let i = 0; i < 1500; i++) {
      const x = Math.random() * W
      const y = Math.random() * H
      ctx.fillStyle = `rgba(90,110,70,${Math.random() * 0.06})`
      ctx.fillRect(x, y, Math.random() * 4 + 2, Math.random() * 4 + 2)
    }
    ctx.globalCompositeOperation = 'source-over'

    // Faint graticule
    ctx.strokeStyle = 'rgba(255,255,255,0.03)'
    ctx.lineWidth = 0.5
    for (let lon = -180; lon <= 180; lon += 30) {
      const [x] = toXY(lon, 0)
      ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, H); ctx.stroke()
    }
    for (let lat = -60; lat <= 60; lat += 30) {
      const [, y] = toXY(0, lat)
      ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke()
    }

    return cv
  }

  // ── Bump map (land slightly raised) ──────────────────────────────────────
  function buildBumpMap() {
    const W = 1024, H = 512
    const cv = document.createElement('canvas')
    cv.width = W; cv.height = H
    const ctx = cv.getContext('2d')
    const toXY = (lon, lat) => [(lon + 180) / 360 * W, (90 - lat) / 180 * H]

    ctx.fillStyle = '#000'
    ctx.fillRect(0, 0, W, H)

    CONTINENTS.forEach(c => {
      ctx.beginPath()
      c.pts.forEach(([lon, lat], i) => {
        const [x, y] = toXY(lon, lat)
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
      })
      ctx.closePath()
      ctx.fillStyle = c.fill === 'ice' ? '#555' : '#777'
      ctx.fill()
    })

    return cv
  }

  // ── Fly to Poland (quaternion slerp) ─────────────────────────────────────
  export function flyToPoland() {
    if (isFlying || pinDropped) return
    autoRotate = false

    flyStartQuat.copy(globeGroup.quaternion)

    const lat = POLAND_LAT * Math.PI / 180
    const lon = POLAND_LON * Math.PI / 180

    // Local outward normal of Poland on the un-rotated sphere
    const polandLocal = new THREE.Vector3(
      Math.cos(lat) * Math.cos(lon),
      Math.sin(lat),
      Math.cos(lat) * Math.sin(lon)
    )

    // Land Poland in the empty zone below the CTA buttons (~75% down the
    // hero), clear of the title, progress card, and milestone chip.
    const target = new THREE.Vector3(0, -0.45, 0.89).normalize()

    flyEndQuat.setFromUnitVectors(polandLocal, target)

    flyStart = performance.now()
    isFlying = true
  }

  function easeInOutCubic(t) {
    return t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2
  }

  // ── Drop Poland pin ──────────────────────────────────────────────────────
  function dropPin() {
    const lat = POLAND_LAT * Math.PI / 180
    const lon = POLAND_LON * Math.PI / 180
    const R = 2.0

    const px = R * Math.cos(lat) * Math.cos(lon)
    const py = R * Math.sin(lat)
    const pz = R * Math.cos(lat) * Math.sin(lon)
    const pos = new THREE.Vector3(px, py, pz)
    const outward = pos.clone().normalize()

    const quat = new THREE.Quaternion()
    quat.setFromUnitVectors(new THREE.Vector3(0, 1, 0), outward)

    pinGroup = new THREE.Group()
    pinGroup.position.copy(pos)
    pinGroup.setRotationFromQuaternion(quat)
    pinGroup.scale.set(0, 0, 0)

    // Shaft
    const shaft = new THREE.Mesh(
      new THREE.CylinderGeometry(0.025, 0.025, 0.36, 16),
      new THREE.MeshStandardMaterial({ color: 0xcc0022, metalness: 0.4, roughness: 0.4 })
    )
    shaft.position.y = 0.3
    pinGroup.add(shaft)

    // Tip (cone pointing into the ground)
    const tip = new THREE.Mesh(
      new THREE.ConeGeometry(0.05, 0.16, 16),
      new THREE.MeshStandardMaterial({ color: 0xaa0018, metalness: 0.3, roughness: 0.5 })
    )
    tip.position.y = 0.08
    tip.rotation.x = Math.PI
    pinGroup.add(tip)

    // Head
    const head = new THREE.Mesh(
      new THREE.SphereGeometry(0.14, 24, 24),
      new THREE.MeshStandardMaterial({
        color: 0xff1f3a,
        metalness: 0.6,
        roughness: 0.2,
        emissive: 0x770014,
        emissiveIntensity: 0.6,
      })
    )
    head.position.y = 0.52
    pinGroup.add(head)

    // Shine
    const shine = new THREE.Mesh(
      new THREE.SphereGeometry(0.04, 8, 8),
      new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.7 })
    )
    shine.position.set(-0.05, 0.55, 0.1)
    pinGroup.add(shine)

    globeGroup.add(pinGroup)

    // Pulse rings tangent to the sphere
    const ringQuat = new THREE.Quaternion()
    ringQuat.setFromUnitVectors(new THREE.Vector3(0, 0, 1), outward)
    const ringPos = pos.clone().multiplyScalar(1.005)

    for (let i = 0; i < 3; i++) {
      const ring = new THREE.Mesh(
        new THREE.RingGeometry(0.06, 0.085, 48),
        new THREE.MeshBasicMaterial({
          color: 0xff2a44,
          side: THREE.DoubleSide,
          transparent: true,
          opacity: 0.85,
        })
      )
      ring.position.copy(ringPos)
      ring.setRotationFromQuaternion(ringQuat)
      ring.userData.phase = i / 3
      pulseRings.push(ring)
      globeGroup.add(ring)
    }

    pinDropped = true
    pinScale = 0
    dispatch('pinDropped')
  }

  // ── Init ─────────────────────────────────────────────────────────────────
  onMount(() => {
    init()
    window.addEventListener('resize', onResize)
    return () => {
      window.removeEventListener('resize', onResize)
      cancelAnimationFrame(animId)
      renderer.dispose()
    }
  })

  function init() {
    const w = container.clientWidth
    const h = container.clientHeight

    scene = new THREE.Scene()

    // Pushed back so the globe occupies ~60% of the viewport height
    camera = new THREE.PerspectiveCamera(38, w / h, 0.1, 1000)
    camera.position.set(0, 0, 8)
    camera.lookAt(0, 0, 0)

    renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
    renderer.setSize(w, h)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    renderer.setClearColor(0x000000, 0)
    container.appendChild(renderer.domElement)

    // Stars
    const starPos = []
    for (let i = 0; i < 4000; i++) {
      const theta = Math.random() * Math.PI * 2
      const phi = Math.acos(2 * Math.random() - 1)
      const r = 30 + Math.random() * 70
      starPos.push(
        r * Math.sin(phi) * Math.cos(theta),
        r * Math.sin(phi) * Math.sin(theta),
        r * Math.cos(phi)
      )
    }
    const starGeo = new THREE.BufferGeometry()
    starGeo.setAttribute('position', new THREE.Float32BufferAttribute(starPos, 3))
    scene.add(new THREE.Points(starGeo, new THREE.PointsMaterial({
      color: 0xffffff,
      size: 0.08,
      transparent: true,
      opacity: 0.7,
      sizeAttenuation: true,
    })))

    // Globe
    globeGroup = new THREE.Group()
    scene.add(globeGroup)

    const earthTex = new THREE.CanvasTexture(buildEarthTexture())
    earthTex.colorSpace = THREE.SRGBColorSpace
    const bumpTex = new THREE.CanvasTexture(buildBumpMap())
    const sphereGeo = new THREE.SphereGeometry(2, 96, 96)
    const sphereMat = new THREE.MeshPhongMaterial({
      map: earthTex,
      bumpMap: bumpTex,
      bumpScale: 0.04,
      shininess: 14,
      specular: new THREE.Color(0x1c3a60),
    })
    globeGroup.add(new THREE.Mesh(sphereGeo, sphereMat))

    // Atmosphere rim
    const atmoGeo = new THREE.SphereGeometry(2.09, 48, 48)
    const atmoMat = new THREE.MeshLambertMaterial({
      color: 0x4d99dd,
      side: THREE.BackSide,
      transparent: true,
      opacity: 0.22,
    })
    globeGroup.add(new THREE.Mesh(atmoGeo, atmoMat))

    // Outer glow
    const glowGeo = new THREE.SphereGeometry(2.25, 32, 32)
    const glowMat = new THREE.MeshBasicMaterial({
      color: 0x3e7dc2,
      transparent: true,
      opacity: 0.05,
      side: THREE.BackSide,
    })
    globeGroup.add(new THREE.Mesh(glowGeo, glowMat))

    // Lights
    scene.add(new THREE.AmbientLight(0x334466, 1.6))
    const sun = new THREE.DirectionalLight(0xfff4dc, 2.1)
    sun.position.set(5, 3, 5)
    scene.add(sun)
    const fill = new THREE.DirectionalLight(0x223355, 0.5)
    fill.position.set(-4, -2, -3)
    scene.add(fill)
    const rim = new THREE.DirectionalLight(0x4488cc, 0.35)
    rim.position.set(0, 0, -5)
    scene.add(rim)

    // Start showing Americas/Atlantic, slight axial tilt for character
    const initial = new THREE.Quaternion().setFromEuler(
      new THREE.Euler(0.18, -Math.PI * 0.8, 0, 'XYZ')
    )
    globeGroup.quaternion.copy(initial)

    animate()
  }

  function animate() {
    animId = requestAnimationFrame(animate)

    if (isFlying && flyStart !== null) {
      const t = Math.min((performance.now() - flyStart) / flyDuration, 1)
      const eased = easeInOutCubic(t)
      globeGroup.quaternion.slerpQuaternions(flyStartQuat, flyEndQuat, eased)

      if (t >= 1) {
        isFlying = false
        flyStart = null
        if (!pinDropped) dropPin()
      }
    } else if (autoRotate) {
      tmpQuat.setFromAxisAngle(new THREE.Vector3(0, 1, 0), 0.0009)
      globeGroup.quaternion.multiply(tmpQuat)
    }

    if (pinGroup && pinScale < 1) {
      pinScale = Math.min(pinScale + 0.04, 1)
      const s = pinScale < 0.7
        ? (pinScale / 0.7) * 1.2
        : 1.2 - ((pinScale - 0.7) / 0.3) * 0.2
      pinGroup.scale.set(s, s, s)
    }

    if (pinDropped) {
      pulseT += 0.018
      pulseRings.forEach(ring => {
        const t = (pulseT * 0.5 + ring.userData.phase) % 1.0
        const s = 1 + t * 5.5
        ring.scale.set(s, s, 1)
        ring.material.opacity = (1 - t) * 0.75
      })
    }

    renderer.render(scene, camera)
  }

  function onResize() {
    if (!container) return
    const w = container.clientWidth, h = container.clientHeight
    camera.aspect = w / h
    camera.updateProjectionMatrix()
    renderer.setSize(w, h)
  }
</script>

<div class="globe-wrap" bind:this={container}></div>

<style>
  .globe-wrap {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
  }
  .globe-wrap :global(canvas) {
    display: block;
    width: 100% !important;
    height: 100% !important;
  }
</style>
