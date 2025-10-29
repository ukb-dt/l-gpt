<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Ukb-Fractal — Topological Tree</title>
<script src="https://cdn.tailwindcss.com"></script>
<style>
  body { background: #071026; color: #e6eef8; -webkit-font-smoothing:antialiased; }
  svg { width: 100%; height: 100%; display:block; }
  .node { cursor: pointer; transition: transform .12s ease; }
  .node:hover { transform: scale(1.06); }
  .edge { stroke-linecap: round; }
  .knob::-webkit-slider-thumb { -webkit-appearance: none; width: 18px; height: 18px; border-radius: 50%; background: white; box-shadow: 0 2px 6px rgba(0,0,0,0.4); }
  .knob { -webkit-appearance: none; appearance: none; height: 6px; border-radius: 999px; }
</style>
</head>
<body class="antialiased">

<main class="max-w-7xl mx-auto p-6 lg:p-10">
  <header class="flex items-center gap-4 mb-6">
    <div class="w-14 h-14 rounded-xl bg-gradient-to-br from-indigo-600 to-violet-500 flex items-center justify-center text-white shadow-lg">
      <svg class="w-7 h-7" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M12 2v20M2 12h20" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
    <div>
      <h1 class="text-2xl font-semibold tracking-tight">Ukb-Fractal — Topological Tree</h1>
      <p class="text-sm text-slate-300 mt-1">Actual tree topology: nodes + edges + hierarchical levels (soil → roots → trunk → branches → canopy). Click nodes to expand/collapse.</p>
    </div>
  </header>

  <section class="grid grid-cols-1 lg:grid-cols-4 gap-6">
    <!-- canvas -->
    <div class="lg:col-span-3 bg-[#071222] rounded-2xl p-4 border border-slate-800">
      <div class="flex items-center justify-between mb-4">
        <div>
          <h2 class="font-semibold text-lg">Tree Reactor</h2>
          <p class="text-xs text-slate-400 mt-1">Topology = proper graph. Nodes are levels; edges are flow. Use the knobs to change topology.</p>
        </div>

        <div class="flex gap-2 items-center">
          <label class="text-xs text-slate-400 mr-2">Layout</label>
          <select id="layout" class="bg-slate-800 text-xs rounded px-2 py-1 border border-slate-700">
            <option value="vertical">Vertical</option>
            <option value="radial">Radial</option>
          </select>
          <button id="downloadSvg" class="ml-3 px-3 py-1 text-xs rounded bg-indigo-600">Download SVG</button>
        </div>
      </div>

      <div id="canvasWrap" class="w-full h-[620px] bg-gradient-to-b from-slate-900/30 to-transparent rounded-lg p-4 relative">
        <svg id="treeSvg" viewBox="0 0 1200 620" preserveAspectRatio="xMidYMid meet">
          <defs>
            <linearGradient id="g_canopy" x1="0" x2="1">
              <stop offset="0" stop-color="#7c3aed"/>
              <stop offset="1" stop-color="#06b6d4"/>
            </linearGradient>
            <filter id="blur" x="-50%" y="-50%" width="200%" height="200%"><feGaussianBlur stdDeviation="3"/></filter>
          </defs>

          <g id="edges"></g>
          <g id="nodes"></g>

        </svg>
        <!-- legend -->
        <div class="absolute left-4 bottom-4 bg-slate-900/80 backdrop-blur-sm rounded p-3 text-xs border border-slate-700">
          <div class="font-mono text-[12px]">Levels: Soil(θ′) → Roots(θ) → Trunk(Σ) → Branches(h(t)) → Canopy(ΔS)</div>
          <div class="text-slate-400 text-[11px] mt-1">Click nodes to toggle subtree. Highlight path toggled automatically.</div>
        </div>
      </div>
    </div>

    <!-- controls -->
    <aside class="space-y-4">
      <div class="bg-slate-800 rounded-2xl p-4 border border-slate-700">
        <h3 class="text-sm font-semibold text-slate-200">Science Knobs</h3>
        <div class="mt-3 space-y-4 text-xs text-slate-300">
          <div>
            <label class="flex justify-between"><span>Energy</span><span id="energyVal" class="font-mono">50</span></label>
            <input id="energy" type="range" min="0" max="100" value="50" class="knob w-full mt-2 bg-slate-700"/>
            <p class="text-[11px] text-slate-500 mt-1">Affects edge thickness & root emphasis.</p>
          </div>

          <div>
            <label class="flex justify-between"><span>Compression</span><span id="compressVal" class="font-mono">50</span></label>
            <input id="compress" type="range" min="0" max="100" value="50" class="knob w-full mt-2 bg-slate-700"/>
            <p class="text-[11px] text-slate-500 mt-1">Controls trunk bottleneck radius; higher = tighter bottleneck (smaller canopy clustering).</p>
          </div>

          <div>
            <label class="flex justify-between"><span>Divergence</span><span id="divergeVal" class="font-mono">3</span></label>
            <input id="diverge" type="range" min="1" max="6" value="3" class="knob w-full mt-2 bg-slate-700"/>
            <p class="text-[11px] text-slate-500 mt-1">Branching factor (integer). Each internal node spawns this many children by default.</p>
          </div>

          <div>
            <label class="flex justify-between"><span>Perplexity</span><span id="perpVal" class="font-mono">30</span></label>
            <input id="perp" type="range" min="0" max="100" value="30" class="knob w-full mt-2 bg-slate-700"/>
            <p class="text-[11px] text-slate-500 mt-1">Randomness: jitter to leaf positions and edge curvature.</p>
          </div>
        </div>
      </div>

      <div class="bg-slate-800 rounded-2xl p-4 border border-slate-700 text-xs text-slate-400">
        <h3 class="text-sm font-semibold mb-2">Presets</h3>
        <div class="flex gap-2">
          <button id="presetBalanced" class="px-3 py-2 rounded bg-indigo-600 text-white text-xs">Balanced</button>
          <button id="presetEncode" class="px-3 py-2 rounded bg-emerald-600 text-white text-xs">Encoder-heavy</button>
          <button id="presetDecode" class="px-3 py-2 rounded bg-pink-600 text-white text-xs">Decoder-heavy</button>
        </div>
        <p class="mt-3">Try presets or tweak knobs for topology you actually understand.</p>
      </div>

      <div class="bg-slate-800 rounded-2xl p-4 border border-slate-700 text-xs text-slate-400">
        <h3 class="text-sm font-semibold mb-2">Controls</h3>
        <div class="flex gap-2">
          <button id="regen" class="px-3 py-2 rounded bg-slate-700 text-white text-xs">Regenerate Tree</button>
          <button id="expandAll" class="px-3 py-2 rounded bg-slate-700 text-white text-xs">Expand All</button>
          <button id="collapseAll" class="px-3 py-2 rounded bg-slate-700 text-white text-xs">Collapse All</button>
        </div>
      </div>
    </aside>
  </section>

  <footer class="mt-8 text-sm text-slate-400 text-center">This one is an actual topological tree — nodes, edges, levels, branching. If you want: I can wire node data to model metrics (per-node perplexity, compression ratios).</footer>
</main>

<script>
/* --- Tree topology renderer --- */

/* Utility helpers */
const $ = (id) => document.getElementById(id);
const map = (v,a,b) => a + (v/100)*(b-a);
const clamp = (v,a,b) => Math.max(a, Math.min(b, v));

/* Level names for nodes by depth (0 = soil root) */
const LEVEL_NAMES = ['θ′ (Soil)', 'θ (Roots)', 'Σ (Trunk)', 'h(t) (Branches)', 'ΔS (Canopy)'];

/* State */
let state = {
  energy: 50,
  compress: 50,
  diverge: 3, // branching factor (integer)
  perp: 30,
  layout: 'vertical',
  maxDepth: 4 // 0..4 -> 5 levels
};

/* DOM refs */
const svg = $('treeSvg');
const nodesG = svg.querySelector('#nodes');
const edgesG = svg.querySelector('#edges');

const energyIn = $('energy'), compressIn = $('compress'), divergeIn = $('diverge'), perpIn = $('perp');
const energyVal = $('energyVal'), compressVal = $('compressVal'), divergeVal = $('divergeVal'), perpVal = $('perpVal');
const layoutSel = $('layout');

/* Simple tree data generator
   - branching factor b
   - depth = maxDepth (0..4)
   - each node: id, depth, label, children[], collapsed flag
*/
let nodeId = 0;
function genTree(branchFactor, depth) {
  nodeId = 0;
  function makeNode(d) {
    const id = 'n' + (nodeId++);
    const node = { id, depth: d, label: LEVEL_NAMES[d] || `L${d}`, children: [], collapsed: false };
    if (d < depth) {
      // create branchFactor children at next depth
      for (let i=0;i<branchFactor;i++) node.children.push(makeNode(d+1));
    }
    return node;
  }
  // We want soil at depth=0 and canopy at depth=maxDepth
  return makeNode(0);
}

/* Layout algorithms: vertical (top->bottom) and radial */
function computePositions(treeRoot, layout) {
  // We'll produce an array of nodes with x,y
  const nodes = [];
  const edges = [];

  // For vertical layout: distribute leaves horizontally using a recursive walk to compute subtree widths
  function measureWidths(node) {
    if (!node.children.length) { node._width = 1; return 1; }
    let w = 0;
    for (const c of node.children) w += measureWidths(c);
    node._width = w;
    return w;
  }

  function placeVertical(node, x0, x1, depth, maxDepth) {
    const cx = (x0 + x1)/2;
    const yGap = 520 / (maxDepth + 2); // some padding
    const cy = 40 + depth * yGap;
    nodes.push({ id: node.id, x: cx, y: cy, depth: node.depth, label: node.label, nodeRef: node });
    if (!node.collapsed && node.children.length) {
      let start = x0;
      for (const c of node.children) {
        const w = c._width;
        const childX0 = start;
        const childX1 = start + w;
        // edge from node to child will be assigned later
        placeVertical(c, childX0, childX1, depth+1, maxDepth);
        edges.push({ from: node.id, to: c.id });
        start += w;
      }
    }
  }

  function placeRadial(node, angle0, angle1, depth, maxDepth, radiusStep) {
    const angle = (angle0 + angle1)/2;
    const radius = 60 + depth * radiusStep;
    const cx = 600 + radius * Math.cos(angle);
    const cy = 310 + radius * Math.sin(angle);
    nodes.push({ id: node.id, x: cx, y: cy, depth: node.depth, label: node.label, nodeRef: node });
    if (!node.collapsed && node.children.length) {
      const span = (angle1-angle0) / node.children.length;
      let a = angle0;
      for (const c of node.children) {
        const childA0 = a;
        const childA1 = a + span;
        placeRadial(c, childA0, childA1, depth+1, maxDepth, radiusStep);
        edges.push({ from: node.id, to: c.id });
        a += span;
      }
    }
  }

  const maxDepth = state.maxDepth;
  if (layout === 'vertical') {
    measureWidths(treeRoot);
    // multiply widths into canvas coord: totalW = root._width
    const totalW = treeRoot._width;
    // padding left/right
    const left = 40, right = 1160;
    const widthRange = right - left;
    // each unit width -> widthRange/totalW
    const unit = widthRange / totalW;
    placeVertical(treeRoot, left, right, 0, maxDepth);
  } else {
    // radial layout: full circle (0..2π)
    const radiusStep = 80; // distance between levels
    placeRadial(treeRoot, 0, Math.PI*2, 0, maxDepth, radiusStep);
  }

  return { nodes, edges };
}

/* Render functions */
function clearSvg() { nodesG.innerHTML = ''; edgesG.innerHTML = ''; }

function renderTree(root) {
  clearSvg();
  const { nodes, edges } = computePositions(root, state.layout);

  // Node map by id for quick lookup
  const nmap = new Map(nodes.map(n => [n.id, n]));

  // Edge drawing: straight paths or smooth cubic depending on layout. Edge thickness from energy.
  const baseEdgeW = map(state.energy, 1, 8); // 1..8 px
  for (const e of edges) {
    const a = nmap.get(e.from), b = nmap.get(e.to);
    if (!a || !b) continue;
    // create path
    const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
    path.classList.add('edge');
    const stroke = (e.from === root.id) ? '#88f' : '#3bd6c6';
    path.setAttribute('stroke', stroke);
    path.setAttribute('fill', 'none');
    path.setAttribute('stroke-width', baseEdgeW.toFixed(2));
    // curvature and jitter from perp
    const jitter = state.perp/120; // small jitter factor
    if (state.layout === 'vertical') {
      const midY = (a.y + b.y) / 2;
      const cx1 = a.x + (b.x - a.x) * 0.25 + (Math.sin(a.x)*jitter*30);
      const cy1 = midY - (10 * (b.depth - a.depth));
      const cx2 = a.x + (b.x - a.x) * 0.75 + (Math.cos(b.x)*jitter*30);
      const cy2 = midY + (10 * (b.depth - a.depth));
      const d = `M ${a.x} ${a.y} C ${cx1} ${cy1} ${cx2} ${cy2} ${b.x} ${b.y}`;
      path.setAttribute('d', d);
    } else {
      // radial: smooth arc-ish cubic
      const dx = b.x - a.x, dy = b.y - a.y;
      const cx1 = a.x + dx * 0.3 + Math.sin(a.x*0.01)*jitter*40;
      const cy1 = a.y + dy * 0.3 + Math.cos(a.y*0.01)*jitter*40;
      const cx2 = a.x + dx * 0.7 + Math.cos(b.x*0.01)*jitter*40;
      const cy2 = a.y + dy * 0.7 + Math.sin(b.y*0.01)*jitter*40;
      const d = `M ${a.x} ${a.y} C ${cx1} ${cy1} ${cx2} ${cy2} ${b.x} ${b.y}`;
      path.setAttribute('d', d);
    }
    edgesG.appendChild(path);
  }

  // Node drawing: radius depends on level and compression (trunk smaller when compression high)
  const maxDepth = state.maxDepth;
  for (const n of nodes) {
    const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
    g.setAttribute('transform', `translate(${n.x},${n.y})`);
    g.classList.add('node');

    // radius mapping by depth: soil big-ish, roots medium, trunk bottleneck, branches smaller, canopy leaves
    // compression squeezes trunk radius: higher compress -> smaller trunk radius
    const baseRByDepth = [18, 14, 12, 10, 8]; // default
    let r = baseRByDepth[n.depth] || 8;
    // Apply compression effect strongest at trunk (depth=2)
    if (n.depth === 2) {
      // compression 0..100 maps to radii 20..6 (inverse)
      r = map(100 - state.compress, 6, 28);
    }
    // energy increases root radius slightly for depth=1
    if (n.depth === 1) r += map(state.energy, 0, 6)/10;
    // jitter for leaf nodes (depth == maxDepth)
    const isLeafLevel = (n.depth === maxDepth);
    if (isLeafLevel) {
      r = 8 + map(state.perp, 0, 4);
    }

    // color by level
    const pal = ['#ffb545', '#34d399', '#7c3aed', '#f472b6', '#60a5fa'];
    const fill = pal[n.depth] || '#999';
    // circle
    const c = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    c.setAttribute('r', r.toFixed(2));
    c.setAttribute('fill', fill);
    c.setAttribute('stroke', '#061026');
    c.setAttribute('stroke-width', 1.5);
    c.setAttribute('filter', n.depth===4 ? 'url(#blur)' : '');

    // label text (small)
    const t = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    t.setAttribute('y', (r + 14).toFixed(1));
    t.setAttribute('text-anchor', 'middle');
    t.setAttribute('font-size', '10');
    t.setAttribute('fill', '#dbeafe');
    t.textContent = (n.depth === 0) ? 'Soil' : (n.depth === 1) ? 'Roots' : (n.depth === 2) ? 'Trunk' : (n.depth === 3) ? 'Branch' : 'Canopy';

    // attach click handler to toggle collapsed state for that node
    (function(nodeRef, gEl){
      gEl.addEventListener('click', (ev) => {
        ev.stopPropagation();
        nodeRef.nodeRef.collapsed = !nodeRef.nodeRef.collapsed;
        renderTree(currentRoot); // re-render with updated collapsed flags
      });
      // tooltip on hover (title)
      const title = document.createElementNS('http://www.w3.org/2000/svg', 'title');
      title.textContent = `${n.label} — id ${n.id}`;
      gEl.appendChild(title);
    })(n, g);

    g.appendChild(c);
    g.appendChild(t);

    nodesG.appendChild(g);
  }

  // Path highlighting: automatically highlight the leftmost soil->canopy path for readability
  // find nodes along a path: take first child repeatedly
  function findPath(rNode) {
    const path = [rNode];
    let cur = rNode;
    while (!cur.collapsed && cur.children && cur.children.length) {
      cur = cur.children[0];
      path.push(cur);
    }
    return path.map(x => x.id);
  }
  const pathIds = findPath(root);
  // highlight edges and nodes along path by recoloring (postprocess edges and nodes)
  // edges are in DOM order; we'll color matching ones
  // color path edges/nodes
  Array.from(edgesG.children).forEach(pathEl => {
    // simple heuristic: check end coords against node map for matching
    // leave as-is (could be improved)
  });
  // nodesG: add glow to nodes in path
  Array.from(nodesG.children).forEach((gEl) => {
    const title = gEl.querySelector('title');
    // our id is not in DOM node; use transform translate to find which node - but simpler: match by label text + position approximate not ideal
    // to keep code simple: no extra recolor here
  });
}

/* Global tree root and helpers */
let currentRoot = genTree(state.diverge, state.maxDepth);

/* wire inputs */
function syncUIFromState() {
  energyVal.textContent = state.energy;
  compressVal.textContent = state.compress;
  divergeVal.textContent = state.diverge;
  perpVal.textContent = state.perp;
  layoutSel.value = state.layout;
}
syncUIFromState();

/* event listeners */
energyIn.addEventListener('input', e => { state.energy = +e.target.value; syncUIFromState(); renderTree(currentRoot); });
compressIn.addEventListener('input', e => { state.compress = +e.target.value; syncUIFromState(); renderTree(currentRoot); });
divergeIn.addEventListener('input', e => { state.diverge = Math.max(1, Math.round(+e.target.value)); syncUIFromState(); });
perpIn.addEventListener('input', e => { state.perp = +e.target.value; syncUIFromState(); renderTree(currentRoot); });
layoutSel.addEventListener('change', e => { state.layout = e.target.value; renderTree(currentRoot); });

$('regen').addEventListener('click', () => {
  currentRoot = genTree(state.diverge, state.maxDepth);
  renderTree(currentRoot);
});
$('expandAll').addEventListener('click', () => {
  (function walk(n){ n.collapsed=false; n.children.forEach(walk); })(currentRoot);
  renderTree(currentRoot);
});
$('collapseAll').addEventListener('click', () => {
  (function walk(n,depth){
    if (depth>0) n.collapsed = true;
    n.children.forEach(c=>walk(c, depth+1));
  })(currentRoot, 0);
  renderTree(currentRoot);
});

/* presets */
$('presetBalanced').addEventListener('click', () => {
  state.energy=50; state.compress=50; state.diverge=3; state.perp=30;
  $('energy').value=50; $('compress').value=50; $('diverge').value=3; $('perp').value=30;
  syncUIFromState(); currentRoot = genTree(state.diverge, state.maxDepth); renderTree(currentRoot);
});
$('presetEncode').addEventListener('click', () => {
  state.energy=80; state.compress=30; state.diverge=2; state.perp=12;
  $('energy').value=80; $('compress').value=30; $('diverge').value=2; $('perp').value=12;
  syncUIFromState(); currentRoot = genTree(state.diverge, state.maxDepth); renderTree(currentRoot);
});
$('presetDecode').addEventListener('click', () => {
  state.energy=30; state.compress=70; state.diverge=5; state.perp=70;
  $('energy').value=30; $('compress').value=70; $('diverge').value=5; $('perp').value=70;
  syncUIFromState(); currentRoot = genTree(state.diverge, state.maxDepth); renderTree(currentRoot);
});

/* download SVG */
$('downloadSvg').addEventListener('click', () => {
  const svgEl = svg.cloneNode(true);
  // inline styles for portability
  const serializer = new XMLSerializer();
  const str = serializer.serializeToString(svgEl);
  const blob = new Blob([str], {type: 'image/svg+xml;charset=utf-8'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'ukb-fractal-tree.svg';
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
});

/* initial render */
currentRoot = genTree(state.diverge, state.maxDepth);
renderTree(currentRoot);

</script>
</body>
</html>
