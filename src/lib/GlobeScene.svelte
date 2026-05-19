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
  let flyDuration = 2400
  let pinDropped = false
  let pinScale = 0
  let autoRotate = true

  const flyStartQuat = new THREE.Quaternion()
  const flyEndQuat = new THREE.Quaternion()
  const tmpQuat = new THREE.Quaternion()

  const POLAND_LAT = 51.9
  const POLAND_LON = 19.5
  const R = 2.0

  // (lat°, lon°) → 3D position on a sphere of `radius`, matching the
  // equirectangular texture mapping used by three.js SphereGeometry. The
  // texture is sampled at x=(lon+180)/360, which puts theta = (lon+180)° in
  // the three.js convention, so z carries a negative sin(lon).
  function latLonToVec3(latDeg, lonDeg, radius = 1) {
    const lat = latDeg * Math.PI / 180
    const lon = lonDeg * Math.PI / 180
    const c = Math.cos(lat)
    return new THREE.Vector3(
      radius * c * Math.cos(lon),
      radius * Math.sin(lat),
      -radius * c * Math.sin(lon)
    )
  }

  // ── Continent outlines (lon, lat) ────────────────────────────────────────
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

    // Eurasia — Atlantic Europe → Arctic Siberia → East Asia → South Asia →
    // Arabia → Levant → Mediterranean Europe. Polygon encloses Poland inland.
    { pts: [
      // Iberia (Cape St Vincent → Galicia)
      [-9.5,37],[-9,39],[-9,41.5],[-8.5,43],[-9,43.5],
      // Cantabrian → Brittany → English Channel
      [-4,43.6],[-2,43.4],[-1.4,46.5],[-4.5,48],[-1.5,49.3],
      // Belgium → Netherlands → German Bight
      [1.5,50.7],[3.5,51.5],[4.5,52.7],[7,53.5],[8.5,53.6],
      // Jutland west → north tip → east
      [8.5,55],[8.3,56.5],[9,57.5],[10.5,57.7],[11,56.3],[12.5,55.6],
      // Baltic south coast: Germany → Poland → Lithuania → Latvia → Estonia
      [13,54.5],[14.5,54.1],[16.5,54.5],[19,54.5],[21,54.6],[21,55.5],
      [23,55],[21,56.5],[24,57.5],[24,59.5],[28,60],[30,60.2],
      // Karelia → White Sea → Arctic Russia → Bering
      [33,64],[36,66],[40,67],[44,68],[50,68],[60,69],
      [70,68],[80,73],[100,76],[115,75],[130,73],[142,73],[150,68],
      [155,62],[148,60],
      // Far East: Sakhalin/Kamchatka simplified → Korea → China east coast
      [142,54],[137,47],[131,43],[126,39],[122,32],
      // South China → Indochina
      [115,23],[110,21],[108,16],[106,11],[104,9],[101,4],
      // Malay → Burma → Bangladesh
      [98,6],[98,12],[94,16],[93,20],[91,22],[89,22],
      // India east → south tip → west coast
      [85,19],[83,16],[80,13],[78,8],[73,16],[70,21],[69,22.5],
      // Pakistan → Iran south → Arabia south
      [67,24],[62,25],[58,24],[56,26],[56,24],[50,25],
      [49,27],[48,30],[45,25],[44,22],[43,16],[44,13],[43,12],
      // Arabia west → Sinai → Levant
      [38,19],[37,22],[36,25],[35,28],[34,30],[34,31],[35,31.5],
      [35,33],[36,35.5],[36,37],
      // Turkey south → Aegean east → Bosphorus
      [35,36.5],[32,36],[30,36.5],[28,37],[27,38.5],[26.5,40],[28,41],
      // Black Sea south → Caucasus
      [33,42],[38,42],[41,41.5],[41,43],[39,43.5],[37,44.5],
      // Crimea / N Black Sea
      [36,45.5],[33,46.2],[31,46.5],[30,45.5],[28.5,45],
      // Bulgaria/Greece east → Aegean
      [28,43.5],[28,41.8],[25,40.5],[23,40.5],[22,39],[23.5,38.5],
      [22,37],[21,36.8],[20,38.5],[19,40.5],
      // Albanian / Croatian / Italian east
      [17.5,42.7],[15,44.2],[13.5,45.7],
      // Italy: heel → toe → tyrrhenian
      [14,41.5],[16.5,41],[18.5,40.2],[17.5,39.5],[16,38],
      [15.5,37.5],[12.5,38.2],[10,42.5],[8.5,44.3],
      // French Riviera → Spain east → Gibraltar
      [3,43.4],[0.5,41],[-1,38],[-5,36.3],[-9.5,37]
    ], fill: 'land' },

    // British Isles
    { pts: [
      [-5.5,50],[-2,50],[0,50.8],[1.5,51.3],[1,52.5],[0,53.5],
      [-1,54.5],[-1.5,55.5],[-2,57.5],[-3.5,58.5],[-5,58.5],
      [-5,56],[-5.5,55],[-4.5,54.5],[-3.5,53],[-4.5,52],
      [-5,51],[-5.5,50]
    ], fill: 'land' },

    // Ireland
    { pts: [[-10,51.5],[-6.5,51.7],[-6,53],[-6.5,54.5],[-8,55.3],[-10,54.5],[-10,52.5],[-10,51.5]], fill: 'land' },

    // Iceland
    { pts: [[-24,63.5],[-22,65],[-18,66.4],[-14,66],[-13,64.5],[-19,63],[-23,63.7],[-24,63.5]], fill: 'land' },

    // Scandinavia (Norway/Sweden west of the main polygon — connects via Karelia)
    // Already enclosed in Eurasia via Baltic and Karelia — no separate poly needed.

    // Africa
    { pts: [
      [-17,21],[-17,16],[-16,14],[-13,11],[-11,8],[-8,6],[-3,5],
      [3,5],[7,4],[9,2],[10,-1],[12,-5],[14,-10],
      [13,-13],[13,-17],[15,-22],[18,-25],[20,-28],[25,-34],
      [29,-33],[33,-29],[35,-25],[40,-22],[40,-16],[40,-12],
      [40,-5],[41,0],[43,4],[47,9],[51,11],[49,7],
      [44,4],[43,10],[42,14],[38,16],[38,19],[36,22],
      [33,27],[31,31],[27,31],[22,32],[15,32],[10,33],
      [3,32],[-2,35],[-7,33],[-12,28],[-17,21]
    ], fill: 'land' },

    // Madagascar
    { pts: [[44,-12],[50,-15],[50,-22],[47,-25],[44,-22],[43,-16],[44,-12]], fill: 'land' },

    // Sumatra
    { pts: [[95,6],[100,5],[105,0],[106,-5],[102,-5],[97,-1],[95,6]], fill: 'land' },

    // Borneo
    { pts: [[109,4],[114,7],[119,5],[119,0],[116,-3],[113,-4],[109,-1],[109,4]], fill: 'land' },

    // Java
    { pts: [[105,-6],[114,-7],[115,-8.5],[108,-8.5],[105,-7],[105,-6]], fill: 'land' },

    // New Guinea
    { pts: [[131,-1],[141,-3],[150,-7],[142,-10],[135,-9],[131,-3],[131,-1]], fill: 'land' },

    // Australia
    { pts: [
      [114,-22],[121,-18],[129,-15],[135,-13],[141,-13],
      [144,-15],[148,-19],[151,-24],[153,-28],[152,-33],
      [149,-37],[144,-38],[140,-38],[136,-34],[128,-32],
      [121,-33],[115,-32],[114,-27],[114,-22]
    ], fill: 'land' },

    // New Zealand (two islands)
    { pts: [[172,-35],[174,-36],[177,-38],[175,-42],[172,-42],[172,-35]], fill: 'land' },
    { pts: [[166,-45],[171,-44],[173,-42],[173,-46],[168,-47],[166,-45]], fill: 'land' },

    // Japan
    { pts: [[130,31],[134,34],[139,35],[141,39],[141,41],[144,43],[145,45],[142,45],[139,41],[136,36],[133,33],[130,31]], fill: 'land' },

    // Sicily
    { pts: [[12.5,38],[15.5,38.3],[15.5,37],[12.5,37.5],[12.5,38]], fill: 'land' },

    // Sri Lanka
    { pts: [[80,6],[82,7],[82,9.5],[80,9],[80,6]], fill: 'land' },

    // Greenland
    { pts: [[-50,60],[-43,60],[-32,65],[-22,72],[-22,78],[-30,82],[-45,82],[-55,78],[-58,72],[-55,65],[-50,60]], fill: 'ice' },

    // Antarctica
    { pts: [[-180,-65],[-120,-70],[-60,-72],[0,-70],[60,-68],[120,-70],[180,-65],[180,-90],[-180,-90],[-180,-65]], fill: 'ice' },
  ]

  // Inland seas / bays drawn over land in ocean color so famous inland water
  // bodies show through (helps Europe look like Europe).
  const INLAND_SEAS = [
    // Black Sea
    [[28,41.5],[33,41.8],[37,41.7],[40.5,42],[41.5,43.2],[40,45],[37.5,45.5],
     [36,46],[33,46.2],[30,46.2],[28,46],[27.7,44],[28,41.5]],
    // Caspian Sea
    [[48,37],[52,37],[54,41],[54,45],[51,47],[49,46],[48.5,42],[48,37]],
    // Red Sea
    [[34,28],[38,28],[43,18],[44,13],[42,13],[38,22],[35,25],[34,28]],
    // Persian Gulf
    [[48,30],[57,25.5],[56,24],[50,28],[48,30]],
    // Hudson Bay
    [[-95,60],[-78,60],[-78,63],[-82,66],[-90,65],[-95,60]],
    // Great Lakes (rough oval)
    [[-92,47],[-76,46],[-76,44],[-85,42],[-92,45],[-92,47]],
    // Baltic proper (the body of water — supplements the south coast)
    [[14,54.5],[18,55],[21,57],[22,59],[20,60],[18,60.5],[15,59],[13,56],[14,54.5]],
    // North Sea bite (between Britain and Denmark) — softens texture
    [[1,52],[8,53.5],[8,57],[2,58],[-1,55],[1,52]],
    // Adriatic Sea (Italy east / Balkans west)
    [[12.5,45.5],[14,44.5],[16,42.5],[18,40.5],[19,40.4],[18.5,41.5],
     [17,43.5],[15,44.8],[13.5,45.6],[12.5,45.5]],
    // Tyrrhenian Sea (Italy west / Sardinia east — rough)
    [[9.5,43.5],[11,42],[13,40.5],[14.5,39],[14,38],[12,38.5],
     [10.5,40],[9.5,42],[9.5,43.5]],
    // Aegean Sea (Greece east / Turkey west)
    [[23,40],[26,40.5],[27,38],[26,36],[24,36],[23,37],[23,40]],
  ]

  // Subtle country/region borders for Europe so Poland reads as "Poland"
  // (interior lines, faintly drawn over land).
  const EUROPE_BORDERS = [
    // Germany / Poland
    [[14.5,54.1],[14.5,52.5],[15,51.5],[14.8,50.9]],
    // Poland / Czechia + Slovakia (southern border)
    [[14.8,50.9],[16.5,50.2],[18.5,49.6],[20,49.4],[22.5,49.1]],
    // Poland / Ukraine + Belarus (eastern border)
    [[22.5,49.1],[23.5,50.5],[23.7,52],[23.2,53.5],[23.2,54.4]],
    // Czech / Austria / Hungary belt
    [[16.5,50.2],[17,48.8],[19,48.5],[22.5,48.5]],
    // France / Germany (Rhine)
    [[7,49],[7.5,48],[7.6,47.5]],
    // Italy north (Alps)
    [[7,44],[10,46],[13,46.5],[13.5,45.7]],
  ]

  // ── Earth canvas texture ─────────────────────────────────────────────────
  function buildEarthTexture() {
    const W = 2048, H = 1024
    const cv = document.createElement('canvas')
    cv.width = W; cv.height = H
    const ctx = cv.getContext('2d')

    const toXY = (lon, lat) => [(lon + 180) / 360 * W, (90 - lat) / 180 * H]

    // Ocean: layered blue with deeper trenches near equator
    const ocean = ctx.createLinearGradient(0, 0, 0, H)
    ocean.addColorStop(0,    '#0e1c30')
    ocean.addColorStop(0.18, '#102a4a')
    ocean.addColorStop(0.42, '#0e3a72')
    ocean.addColorStop(0.5,  '#0f4080')
    ocean.addColorStop(0.58, '#0e3a72')
    ocean.addColorStop(0.82, '#102a4a')
    ocean.addColorStop(1,    '#0e1c30')
    ctx.fillStyle = ocean
    ctx.fillRect(0, 0, W, H)

    // Cloud-cell ocean texture
    for (let i = 0; i < 800; i++) {
      const x = Math.random() * W
      const y = Math.random() * H
      const r = Math.random() * 100 + 30
      ctx.fillStyle = `rgba(40,95,160,${Math.random() * 0.06})`
      ctx.beginPath()
      ctx.arc(x, y, r, 0, Math.PI * 2)
      ctx.fill()
    }

    const tracePath = (pts) => {
      ctx.beginPath()
      pts.forEach(([lon, lat], i) => {
        const [x, y] = toXY(lon, lat)
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
      })
      ctx.closePath()
    }

    const fillContinent = (pts, fillType) => {
      tracePath(pts)
      if (fillType === 'ice') {
        const ice = ctx.createLinearGradient(0, 0, 0, H)
        ice.addColorStop(0, '#f1f6fa')
        ice.addColorStop(0.5, '#d7e3ec')
        ice.addColorStop(1, '#f1f6fa')
        ctx.fillStyle = ice
        ctx.fill()
        ctx.strokeStyle = 'rgba(190,210,225,0.55)'
        ctx.lineWidth = 1
        ctx.stroke()
      } else {
        const land = ctx.createLinearGradient(0, 0, 0, H)
        land.addColorStop(0,    '#4a5a36')
        land.addColorStop(0.18, '#5a7240')
        land.addColorStop(0.32, '#6f8a46')
        land.addColorStop(0.42, '#80994b')
        land.addColorStop(0.5,  '#8da14d')
        land.addColorStop(0.58, '#80994b')
        land.addColorStop(0.68, '#6f8a46')
        land.addColorStop(0.82, '#5a7240')
        land.addColorStop(1,    '#4a5a36')
        ctx.fillStyle = land
        ctx.fill()
        ctx.strokeStyle = 'rgba(38,52,24,0.65)'
        ctx.lineWidth = 1.5
        ctx.stroke()
      }
    }

    CONTINENTS.forEach(c => fillContinent(c.pts, c.fill))

    // Deserts (Sahara/Arabia, Outback, Gobi, Atacama, Kalahari)
    const deserts = [
      { pts: [[-15,32],[40,32],[50,22],[40,14],[-15,18]], color: 'rgba(210,175,100,0.5)' },
      { pts: [[42,20],[55,20],[55,15],[44,14]],            color: 'rgba(210,175,100,0.4)' },
      { pts: [[115,-20],[145,-22],[145,-32],[120,-30]],    color: 'rgba(205,165,95,0.4)' },
      { pts: [[80,48],[110,48],[110,38],[80,38]],          color: 'rgba(190,160,90,0.22)' },
      { pts: [[18,-19],[24,-19],[24,-28],[18,-26]],        color: 'rgba(205,170,100,0.35)' },
      { pts: [[-72,-20],[-69,-20],[-69,-28],[-72,-26]],    color: 'rgba(195,160,90,0.35)' },
    ]
    deserts.forEach(({ pts, color }) => {
      tracePath(pts)
      ctx.fillStyle = color
      ctx.fill()
    })

    // Inland seas / bays — paint with ocean tone
    ctx.fillStyle = '#0f3d72'
    INLAND_SEAS.forEach(pts => {
      tracePath(pts)
      ctx.fill()
    })
    // Slight rim on the inland water
    ctx.strokeStyle = 'rgba(20,40,80,0.35)'
    ctx.lineWidth = 1
    INLAND_SEAS.forEach(pts => { tracePath(pts); ctx.stroke() })

    // Faint European borders so the region reads better
    ctx.strokeStyle = 'rgba(40,55,25,0.45)'
    ctx.lineWidth = 0.9
    EUROPE_BORDERS.forEach(pts => {
      ctx.beginPath()
      pts.forEach(([lon, lat], i) => {
        const [x, y] = toXY(lon, lat)
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
      })
      ctx.stroke()
    })

    // Highlight Poland's land area subtly so the pin lands on something
    // visibly distinct rather than a generic green smear.
    ctx.save()
    ctx.beginPath()
    const polandOutline = [
      [14.5,54.1],[16.5,54.5],[19,54.5],[21,54.6],[23.2,54.4],
      [23.5,52.5],[23.7,51],[24,50.5],[22.5,49.1],[20,49.4],
      [18.5,49.6],[16.5,50.2],[14.8,50.9],[14.5,52.5],[14.5,54.1]
    ]
    polandOutline.forEach(([lon, lat], i) => {
      const [x, y] = toXY(lon, lat)
      i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
    })
    ctx.closePath()
    ctx.fillStyle = 'rgba(220,165,70,0.18)'
    ctx.fill()
    ctx.strokeStyle = 'rgba(60,40,15,0.55)'
    ctx.lineWidth = 1.2
    ctx.stroke()
    ctx.restore()

    // Subtle terrain noise
    ctx.globalCompositeOperation = 'overlay'
    for (let i = 0; i < 2200; i++) {
      const x = Math.random() * W
      const y = Math.random() * H
      ctx.fillStyle = `rgba(90,110,70,${Math.random() * 0.07})`
      ctx.fillRect(x, y, Math.random() * 4 + 2, Math.random() * 4 + 2)
    }
    ctx.globalCompositeOperation = 'source-over'

    // Faint graticule
    ctx.strokeStyle = 'rgba(255,255,255,0.035)'
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
      ctx.fillStyle = c.fill === 'ice' ? '#555' : '#888'
      ctx.fill()
    })

    // Punch inland seas back to "low"
    ctx.fillStyle = '#000'
    INLAND_SEAS.forEach(pts => {
      ctx.beginPath()
      pts.forEach(([lon, lat], i) => {
        const [x, y] = toXY(lon, lat)
        i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y)
      })
      ctx.closePath()
      ctx.fill()
    })

    return cv
  }

  // ── Fly to Poland (quaternion slerp) ─────────────────────────────────────
  export function flyToPoland() {
    if (isFlying || pinDropped) return
    autoRotate = false

    flyStartQuat.copy(globeGroup.quaternion)

    // Poland on the un-rotated sphere (matches texture mapping)
    const polandLocal = latLonToVec3(POLAND_LAT, POLAND_LON, 1)

    // Place Poland in the upper-right of the visible disk so it sits in the
    // empty space beside the hero title / progress card rather than behind
    // them. +X = screen right, +Y = screen up, +Z = toward camera.
    const target = new THREE.Vector3(0.72, 0.46, 0.52).normalize()

    flyEndQuat.setFromUnitVectors(polandLocal, target)

    flyStart = performance.now()
    isFlying = true
  }

  function easeInOutCubic(t) {
    return t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2
  }

  // ── Drop Poland pin ──────────────────────────────────────────────────────
  function dropPin() {
    const pos = latLonToVec3(POLAND_LAT, POLAND_LON, R)
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
        emissiveIntensity: 0.7,
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
      opacity: 0.75,
      sizeAttenuation: true,
    })))

    // Globe
    globeGroup = new THREE.Group()
    scene.add(globeGroup)

    const earthTex = new THREE.CanvasTexture(buildEarthTexture())
    earthTex.colorSpace = THREE.SRGBColorSpace
    earthTex.anisotropy = 8
    const bumpTex = new THREE.CanvasTexture(buildBumpMap())
    const sphereGeo = new THREE.SphereGeometry(R, 96, 96)
    const sphereMat = new THREE.MeshPhongMaterial({
      map: earthTex,
      bumpMap: bumpTex,
      bumpScale: 0.04,
      shininess: 18,
      specular: new THREE.Color(0x214266),
    })
    globeGroup.add(new THREE.Mesh(sphereGeo, sphereMat))

    // Atmosphere rim
    const atmoGeo = new THREE.SphereGeometry(R * 1.045, 48, 48)
    const atmoMat = new THREE.MeshLambertMaterial({
      color: 0x4d99dd,
      side: THREE.BackSide,
      transparent: true,
      opacity: 0.25,
    })
    globeGroup.add(new THREE.Mesh(atmoGeo, atmoMat))

    // Outer glow
    const glowGeo = new THREE.SphereGeometry(R * 1.13, 32, 32)
    const glowMat = new THREE.MeshBasicMaterial({
      color: 0x3e7dc2,
      transparent: true,
      opacity: 0.06,
      side: THREE.BackSide,
    })
    globeGroup.add(new THREE.Mesh(glowGeo, glowMat))

    // Lights
    scene.add(new THREE.AmbientLight(0x334466, 1.6))
    const sun = new THREE.DirectionalLight(0xfff4dc, 2.2)
    sun.position.set(5, 3, 5)
    scene.add(sun)
    const fill = new THREE.DirectionalLight(0x223355, 0.55)
    fill.position.set(-4, -2, -3)
    scene.add(fill)
    const rim = new THREE.DirectionalLight(0x4488cc, 0.4)
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
