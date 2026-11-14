---
layout: post
title: "Understanding Dynamic Hedging"
description: "Working through Taleb's Dynamic Hedging as a programmer, with a small simulator showing why hedging in discrete time never matches the textbook."
date: 2026-11-14 07:30:00 -0600
categories: [general]
tags: [finance, options, python, javascript]
---

*A note before starting: I received significant assistance from Anthropic's Claude on this post, especially in creating the interactive JavaScript simulator. Since the subject is outside my area of expertise, I leaned on AI to help me write and understand the concepts explained below. In recent months I've decided to be more permissive about using AI as a tool for gaining knowledge outside my area of expertise, and this article is largely an experiment in diversifying the content here a little.*

<!-- TODO(graham): replace this opening with your own reason for picking the book up. -->
I've been slowly working through Nassim Taleb's *Dynamic Hedging* [1], which he wrote in 1997, years before the books he's actually known for. I must admit I'm not a trader and have never run an options book (a trader's whole portfolio of positions), so take this as a programmer's notes on a trader's manual and not as advice on anything.

What kept me reading is that the book is about the gap between a model and the thing the model describes. That's a gap that anyone who has shipped software should recognize.

## Five Minutes of Vocabulary

Finance has a lot of jargon, and most of it turns out to be short names for simple things. If you already know what a call option is, skip ahead. If not, this is everything the rest of the post leans on.

**A call option** is a contract that gives you the right, but not the obligation, to buy a stock at a fixed price on a future date. The fixed price is the **strike** and the date is the **expiry**. The stock it refers to is called the **underlying**. What you pay for the contract up front is the **premium**.

The whole contract fits in one line:

```python
payoff_at_expiry = max(stock_price - strike, 0)
```

Say the stock is at 100 and I pay 4 for a three-month call with a strike of 100. If the stock finishes at 110, I use my right to buy at 100, the contract is worth 10 and I'm up 6. If it finishes at 90, I let the contract expire, it's worth nothing and I'm down the 4 I paid. My loss is capped and my gain isn't, which is what I paid the premium for. A strike equal to the current price, like this one, is called **at-the-money**.

**Long and short.** Being long something means I own it and make money when it goes up. Being short means I've sold something I borrowed, so I make money when it goes down. Being **flat** means my positions cancel out and I don't care which way the price goes.

**Hedging** is taking a second position that loses when the first one wins, so that together they're flat. If I own a call (which gains when the stock rises), I can short some of the stock (which loses when it rises) against it.

**Volatility** is how much the price jumps around: the standard deviation of its returns, scaled to a year. A volatility of 20% works out to a typical daily move of about 1.26%, because there are roughly 252 trading days in a year and standard deviations scale with the square root of time (20% / √252). There are two flavors, and the difference between them is what this whole post is about:

- **Realized volatility** is what the stock actually did. You can only measure it afterwards.
- **Implied volatility** is the market's guess, read backwards out of option prices. The pricing formula takes a volatility and returns a price; given a price, you can solve for the volatility that would produce it.

**The Greeks** are the partial derivatives of the option's price with respect to its inputs. If the pricing formula is a function, the Greeks are its gradient:

| Greek | Derivative with respect to | In plain words |
|-------|---------------------------|----------------|
| Delta | stock price | how much the option moves when the stock moves by 1 |
| Gamma | stock price, twice | how fast delta itself changes |
| Theta | time | how much value the option loses per day just by aging |
| Vega | volatility | how much the option moves when implied volatility moves |

Two last abbreviations: **P&L** is profit and loss, the running total of what a position has made or lost, and a **bp** (basis point) is a hundredth of a percent, so a 5 bps trading cost on 10,000 of stock is 5.

## The Textbook Version

The Black-Scholes argument [2] goes roughly like this: hold an option, trade the underlying against it continuously, and the two positions cancel each other. What's left is riskless, a riskless position has to earn the risk-free rate (what you'd get for parking the money in government bills), and that pins down the option's price.

For this to work you need a few things:

1. You can trade continuously.
2. Trading costs nothing.
3. Volatility is constant and you know what it is.
4. The price never jumps.

None of these hold for somebody actually running a position. As I read him, Taleb doesn't throw the formula away over this; he treats it as a shared language for quoting prices and then spends several hundred pages on everything the formula leaves behind. Hedging is discrete, it costs money, and the volatility you plugged in is a guess. The job is managing the residue.

The formula itself is small. Here it is for a call option with interest rates set to zero, which is all the Python this post needs and uses nothing outside the standard library:

```python
from math import erf, exp, log, pi, sqrt


def norm_cdf(x):
    return 0.5 * (1 + erf(x / sqrt(2)))


def bs_call(S, K, tau, vol):
    """Black-Scholes call with zero rates. Returns (price, delta)."""
    if tau <= 0:
        return max(S - K, 0.0), float(S > K)
    sd = vol * sqrt(tau)
    d1 = (log(S / K) + 0.5 * sd * sd) / sd
    return S * norm_cdf(d1) - K * norm_cdf(d1 - sd), norm_cdf(d1)
```

`S` is the stock price, `K` the strike, `tau` the time to expiry in years and `vol` the annualized volatility. It's a pure function of four numbers, and three of them you can read off a screen. The fourth, `vol`, is the one you have to make up.

## Delta Is a Hedge Ratio, Not a Fact

Delta is how much the option's value moves when the underlying moves by one. If I'm long a call with a delta of 0.5, I short half a share against it and for a small move I'm flat.

The catch is that delta isn't a property of the option sitting out in the world. It's an output of a model; change the volatility you feed in and you get a different delta for the same contract:

```python
for vol in (0.10, 0.20, 0.40):
    price, delta = bs_call(S=105, K=100, tau=0.25, vol=vol)
    print(f"vol={vol:.0%}  price={price:5.2f}  delta={delta:.2f}")
```

```text
vol=10%  price= 5.45  delta=0.84
vol=20%  price= 7.06  delta=0.70
vol=40%  price=10.91  delta=0.63
```

Same contract, same stock price, and the "correct" hedge is anywhere from 63 to 84 shares per hundred depending on a parameter nobody can observe. Delta also moves on its own as time passes and as the price moves, which is the whole reason the hedge has to be *dynamic*: the number you hedged with this morning is already stale.

## Gamma and Theta: The Rent You Pay

Gamma is how fast delta changes. If I'm long an option and hedged, a rally makes my delta rise, so I sell more shares to get flat; a selloff makes it fall, so I buy some back. I'm mechanically selling high and buying low, and every rebalance locks in a little money.

Of course that isn't free. The option loses value every day it sits there, which is theta. Over one hedging interval the two net out to approximately:

```text
P&L  ≈  ½ · Γ · S² · [ (ΔS/S)² − σ² · Δt ]
```

Here Γ is gamma, S is the stock price, ΔS is how far it moved over the interval, σ is the implied volatility and Δt is the length of the interval in years. The first term in the brackets is what the price actually did, squared. The second is what the implied volatility σ said it would do. So a hedged option position is a bet on realized volatility against implied volatility, and the direction of the market drops out.

It also gives a handy rule of thumb. With a stock at 100 and implied volatility of 20%, the break-even daily move is about `100 × 0.20 / √252 ≈ 1.26`. If I'm long gamma (I own the option, so big moves in either direction help me) and the stock moves more than that, I made money that day; less, and I paid rent for nothing.

I didn't trust the approximation until I checked it against the pricing function. Buy the call, short delta shares, let one trading day pass with the stock moving by some amount, and compare what the position actually made to what the formula predicts:

```python
S, K, tau, vol, dt = 100, 100, 0.25, 0.20, 1 / 252
price, delta = bs_call(S, K, tau, vol)

sd = vol * sqrt(tau)
d1 = (log(S / K) + 0.5 * sd * sd) / sd
gamma = exp(-d1 * d1 / 2) / sqrt(2 * pi) / (S * sd)

for move in (0.0, 0.5, 1.26, 2.0, 3.0):
    exact = bs_call(S + move, K, tau - dt, vol)[0] - price - delta * move
    approx = 0.5 * gamma * S**2 * ((move / S) ** 2 - vol**2 * dt)
    print(f"move={move:4.2f}  exact={exact:+.3f}  formula={approx:+.3f}")
```

```text
move=0.00  exact=-0.032  formula=-0.032
move=0.50  exact=-0.027  formula=-0.027
move=1.26  exact=-0.000  formula=+0.000
move=2.00  exact=+0.047  formula=+0.048
move=3.00  exact=+0.145  formula=+0.148
```

A day where nothing happens costs about three cents of rent. A move of 1.26 breaks even, as promised, and past that the payoff grows with the square of the move. The sign of the move doesn't appear anywhere; a drop of 3 pays the same as a rally of 3.

## A Simulator

Reading that formula didn't make it click for me; watching it did. Below is a long 3-month at-the-money call (strike 100, rates at zero), bought and delta-hedged at the implied volatility while the stock actually moves at the realized volatility.

{% raw %}
<style>
.dh-sim { border: 1px solid rgba(128,128,128,.35); border-radius: 6px; padding: 14px 16px; margin: 0 0 24px; }
.dh-sim .dh-controls { display: grid; grid-template-columns: repeat(auto-fit, minmax(210px, 1fr)); gap: 10px 22px; margin-bottom: 10px; }
.dh-sim label { display: block; font-size: 14px; }
.dh-sim output { float: right; font-variant-numeric: tabular-nums; font-weight: 600; }
.dh-sim input[type=range] { display: block; width: 100%; margin: 4px 0 0; }
.dh-sim button { font: inherit; font-size: 14px; padding: 4px 12px; border: 1px solid rgba(128,128,128,.6); border-radius: 4px; background: transparent; color: inherit; cursor: pointer; }
.dh-sim button:hover { background: rgba(128,128,128,.15); }
.dh-sim h4 { font-size: 14px; margin: 14px 0 2px; font-weight: 600; }
.dh-sim canvas { display: block; width: 100%; height: 170px; }
.dh-sim .dh-read { font-size: 13px; font-variant-numeric: tabular-nums; margin: 4px 0 0; opacity: .85; }
</style>
<div class="dh-sim" id="dh-sim">
  <div class="dh-controls">
    <label>Implied volatility <output id="dh-iv-o"></output><input type="range" id="dh-iv" min="5" max="60" step="1" value="20"></label>
    <label>Realized volatility <output id="dh-rv-o"></output><input type="range" id="dh-rv" min="5" max="60" step="1" value="20"></label>
    <label>Rehedges until expiry <output id="dh-n-o"></output><input type="range" id="dh-n" min="0" max="9" step="1" value="5"></label>
    <label>Cost per trade (bps) <output id="dh-tc-o"></output><input type="range" id="dh-tc" min="0" max="20" step="1" value="0"></label>
  </div>
  <button type="button" id="dh-new">New path</button>
  <h4>Stock price (dots are rehedges)</h4>
  <canvas id="dh-spot" role="img" aria-label="Simulated stock price path with the strike marked"></canvas>
  <h4>Running P&amp;L of the hedged position, this path</h4>
  <canvas id="dh-pnl" role="img" aria-label="Running profit and loss of the delta-hedged option on this path"></canvas>
  <p class="dh-read" id="dh-read1"></p>
  <h4>Final P&amp;L across 400 paths at these settings</h4>
  <canvas id="dh-hist" role="img" aria-label="Histogram of final profit and loss across 400 simulated paths"></canvas>
  <p class="dh-read" id="dh-read2"></p>
</div>
<script>
(function () {
  var root = document.getElementById('dh-sim');
  if (!root) return;
  var S0 = 100, K = 100, T = 0.25, FINE = 1260, PATHS = 400;
  var GRID = [1, 3, 6, 12, 21, 63, 126, 252, 630, 1260];
  var BLUE = '#2a7ae2', ORANGE = '#d9822b';
  var seed = 7;
  function el(id) { return document.getElementById(id); }
  function rng(a) {
    return function () {
      a = (a + 0x6D2B79F5) | 0;
      var t = Math.imul(a ^ (a >>> 15), 1 | a);
      t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
      return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
    };
  }
  function normals(n, s) {
    var r = rng(s), out = new Float64Array(n);
    for (var i = 0; i < n; i += 2) {
      var m = Math.sqrt(-2 * Math.log(1 - r())), a = 2 * Math.PI * r();
      out[i] = m * Math.cos(a);
      if (i + 1 < n) out[i + 1] = m * Math.sin(a);
    }
    return out;
  }
  function cdf(x) {
    var t = 1 / (1 + 0.2316419 * Math.abs(x));
    var p = 0.3989422804014327 * Math.exp(-x * x / 2) * t * (0.31938153 + t * (-0.356563782 + t * (1.781477937 + t * (-1.821255978 + t * 1.330274429))));
    return x > 0 ? 1 - p : p;
  }
  // Black-Scholes call with zero rates: returns [price, delta].
  function call(S, tau, vol) {
    if (tau <= 0) return [Math.max(S - K, 0), S > K ? 1 : 0];
    var sd = vol * Math.sqrt(tau), d1 = (Math.log(S / K) + 0.5 * sd * sd) / sd;
    return [S * cdf(d1) - K * cdf(d1 - sd), cdf(d1)];
  }
  // Geometric Brownian motion with zero drift, sampled at z.length even steps.
  function gbm(z, vol) {
    var n = z.length, h = T / n, path = new Float64Array(n + 1), s = S0;
    var drift = -0.5 * vol * vol * h, diff = vol * Math.sqrt(h);
    path[0] = s;
    for (var i = 0; i < n; i++) { s *= Math.exp(drift + diff * z[i]); path[i + 1] = s; }
    return path;
  }
  // Long one call, short delta shares, rebalanced at every point of `path`.
  function hedge(path, iv, tc) {
    var n = path.length - 1, dt = T / n, o = call(path[0], T, iv);
    var delta = o[1], costs = delta * path[0] * tc;
    var cash = -o[0] + delta * path[0] - costs;
    var pnl = new Float64Array(n + 1);
    pnl[0] = -costs;
    for (var i = 1; i <= n; i++) {
      var S = path[i], c;
      o = call(S, i === n ? 0 : T - i * dt, iv);
      if (i < n) {
        c = Math.abs(o[1] - delta) * S * tc;
        cash += (o[1] - delta) * S - c;
        delta = o[1];
      } else {
        c = delta * S * tc;
        cash -= c;
      }
      costs += c;
      pnl[i] = cash + o[0] - delta * S;
    }
    return { pnl: pnl, costs: costs, premium: call(path[0], T, iv)[0] };
  }
  function sub(path, n) {
    var k = (path.length - 1) / n, out = new Float64Array(n + 1);
    for (var i = 0; i <= n; i++) out[i] = path[i * k];
    return out;
  }
  function canvas(c) {
    var dpr = window.devicePixelRatio || 1, w = c.clientWidth || 640, h = c.clientHeight || 170;
    c.width = Math.round(w * dpr); c.height = Math.round(h * dpr);
    var g = c.getContext('2d');
    g.setTransform(dpr, 0, 0, dpr, 0, 0);
    g.clearRect(0, 0, w, h);
    var cs = window.getComputedStyle(root);
    g.font = '11px ' + cs.fontFamily;
    return { g: g, w: w, h: h, fg: cs.color, l: 46, r: 8, t: 8, b: 18 };
  }
  function fmt(v) { v = Math.round(v * 100) / 100; return (v > 0 ? '+' : v < 0 ? '−' : '') + Math.abs(v).toFixed(2); }
  function lineChart(c, ys, o) {
    var v = canvas(c), g = v.g, n = ys.length - 1, lo = o.ref, hi = o.ref, i;
    for (i = 0; i <= n; i++) { if (ys[i] < lo) lo = ys[i]; if (ys[i] > hi) hi = ys[i]; }
    var pad = Math.max((hi - lo) * 0.1, o.minPad); lo -= pad; hi += pad;
    var pw = v.w - v.l - v.r, ph = v.h - v.t - v.b;
    function X(i) { return v.l + pw * i / n; }
    function Y(y) { return v.t + ph * (hi - y) / (hi - lo); }
    g.strokeStyle = v.fg; g.fillStyle = v.fg; g.lineWidth = 1;
    g.globalAlpha = 0.35; g.setLineDash([4, 4]);
    g.beginPath(); g.moveTo(v.l, Y(o.ref)); g.lineTo(v.w - v.r, Y(o.ref)); g.stroke();
    g.setLineDash([]); g.globalAlpha = 0.75;
    g.textAlign = 'right'; g.textBaseline = 'middle';
    g.fillText(o.label(hi), v.l - 6, v.t + 5);
    g.fillText(o.label(lo), v.l - 6, v.h - v.b - 5);
    if (Math.abs(Y(o.ref) - v.t) > 16 && Math.abs(Y(o.ref) - (v.h - v.b)) > 16) g.fillText(o.label(o.ref), v.l - 6, Y(o.ref));
    g.textBaseline = 'alphabetic'; g.textAlign = 'left'; g.fillText('today', v.l, v.h - 4);
    g.textAlign = 'right'; g.fillText('expiry', v.w - v.r, v.h - 4);
    g.globalAlpha = 1; g.strokeStyle = o.color; g.lineWidth = 1.5; g.lineJoin = 'round';
    g.beginPath();
    for (i = 0; i <= n; i++) { if (i) g.lineTo(X(i), Y(ys[i])); else g.moveTo(X(i), Y(ys[i])); }
    g.stroke();
    if (o.marks && o.marks <= 63) {
      g.fillStyle = o.color;
      for (i = 0; i < o.marks; i++) { var j = i * n / o.marks; g.beginPath(); g.arc(X(j), Y(ys[j]), 2.5, 0, 6.2832); g.fill(); }
    }
  }
  function histogram(c, xs, mean) {
    var v = canvas(c), g = v.g, bins = 41, counts = new Array(bins), i, top = 0;
    var sorted = Array.prototype.slice.call(xs).sort(function (a, b) { return a - b; });
    var lo = Math.floor(Math.min(-3, sorted[4])), hi = Math.ceil(Math.max(3, sorted[sorted.length - 5]));
    for (i = 0; i < bins; i++) counts[i] = 0;
    for (i = 0; i < xs.length; i++) {
      var b = Math.floor((xs[i] - lo) / (hi - lo) * bins);
      counts[b < 0 ? 0 : b >= bins ? bins - 1 : b]++;
    }
    for (i = 0; i < bins; i++) if (counts[i] > top) top = counts[i];
    var pw = v.w - v.l - v.r, ph = v.h - v.t - v.b, bw = pw / bins;
    function X(x) { return v.l + pw * (x - lo) / (hi - lo); }
    g.fillStyle = BLUE;
    for (i = 0; i < bins; i++) { var bh = ph * counts[i] / top; g.fillRect(v.l + i * bw + 1, v.t + ph - bh, bw - 2, bh); }
    g.strokeStyle = v.fg; g.fillStyle = v.fg; g.lineWidth = 1;
    g.globalAlpha = 0.35; g.setLineDash([4, 4]);
    g.beginPath(); g.moveTo(X(0), v.t); g.lineTo(X(0), v.t + ph); g.stroke();
    g.setLineDash([]); g.globalAlpha = 0.75; g.textBaseline = 'alphabetic';
    g.textAlign = 'left'; g.fillText(fmt(lo), v.l, v.h - 4);
    g.textAlign = 'center'; g.fillText('0', X(0), v.h - 4);
    g.textAlign = 'right'; g.fillText(fmt(hi), v.w - v.r, v.h - 4);
    g.globalAlpha = 1; g.strokeStyle = ORANGE; g.lineWidth = 2;
    var mx = Math.min(Math.max(X(mean), v.l), v.w - v.r);
    g.beginPath(); g.moveTo(mx, v.t); g.lineTo(mx, v.t + ph); g.stroke();
  }
  var state = null;
  function compute() {
    var iv = +el('dh-iv').value / 100, rv = +el('dh-rv').value / 100;
    var n = GRID[+el('dh-n').value], tc = +el('dh-tc').value / 10000;
    var fine = gbm(normals(FINE, seed), rv), one = hedge(sub(fine, n), iv, tc);
    var finals = new Float64Array(PATHS), sum = 0, sq = 0, p;
    for (p = 0; p < PATHS; p++) {
      var r = hedge(gbm(normals(n, 1000 + p), rv), iv, tc);
      finals[p] = r.pnl[n]; sum += finals[p];
    }
    var mean = sum / PATHS;
    for (p = 0; p < PATHS; p++) sq += (finals[p] - mean) * (finals[p] - mean);
    var sd0 = iv * Math.sqrt(T), d1 = 0.5 * sd0;
    var vega = S0 * 0.3989422804014327 * Math.exp(-d1 * d1 / 2) * Math.sqrt(T);
    state = { iv: iv, rv: rv, n: n, tc: tc, fine: fine, one: one, finals: finals, mean: mean, sd: Math.sqrt(sq / (PATHS - 1)), vega: vega };
    return state;
  }
  function draw() {
    var s = state, days = 63 / s.n;
    el('dh-iv-o').textContent = Math.round(s.iv * 100) + '%';
    el('dh-rv-o').textContent = Math.round(s.rv * 100) + '%';
    el('dh-n-o').textContent = s.n + (days === 1 ? ' (daily)' : days > 1 ? ' (every ' + (days % 1 ? days.toFixed(1) : days) + ' days)' : ' (' + Math.round(1 / days) + ' per day)');
    el('dh-tc-o').textContent = Math.round(s.tc * 10000);
    lineChart(el('dh-spot'), s.fine, { ref: K, minPad: 1, color: BLUE, marks: s.n, label: function (y) { return y.toFixed(0); } });
    lineChart(el('dh-pnl'), s.one.pnl, { ref: 0, minPad: 0.25, color: ORANGE, label: fmt });
    el('dh-read1').textContent = 'Premium paid ' + s.one.premium.toFixed(2) + ' · final P&L ' + fmt(s.one.pnl[s.n]) + ' · trading costs ' + s.one.costs.toFixed(2);
    histogram(el('dh-hist'), s.finals, s.mean);
    el('dh-read2').textContent = 'Mean ' + fmt(s.mean) + ' (orange line) · standard deviation ' + s.sd.toFixed(2) + ' · vega × (realized − implied) ≈ ' + fmt(s.vega * (s.rv - s.iv));
  }
  function update() { compute(); draw(); }
  ['dh-iv', 'dh-rv', 'dh-n', 'dh-tc'].forEach(function (id) { el(id).addEventListener('input', update); });
  el('dh-new').addEventListener('click', function () { seed++; update(); });
  var pending = false;
  window.addEventListener('resize', function () {
    if (pending) return;
    pending = true;
    window.requestAnimationFrame(function () { pending = false; if (state) draw(); });
  });
  update();
})();
</script>
<noscript><p><em>The simulator needs JavaScript. The four experiments below describe what it shows.</em></p></noscript>
{% endraw %}

A few things worth trying:

**1. Hedge more often.** Leave both volatilities at 20% and drag the rehedge slider from 1 up to 1260. The average P&L stays near zero the whole way; what collapses is the spread around it. Hedging frequency doesn't change what you expect to make, it changes how much luck is involved. The spread falls with roughly the square root of the number of rehedges [3], so you have to hedge four times as often to cut the noise in half.

**2. Be wrong about volatility.** Set realized to 30% and leave implied at 20%. The whole distribution shifts right by about vega times the difference. I bought volatility at 20, it came in at 30, and the hedging harvested the gap. Flip them and it's a slow bleed.

**3. Same volatility, different outcome.** Put the rehedges at 12 and hit *New path* several times. The inputs haven't changed but the P&L has. Gamma is concentrated near the strike and close to expiry, so *where* and *when* the stock does its moving matters as much as how much it moves. A path that sits near 100 in the last two weeks is a very different animal from one that wandered off to 115 in the first month.

**4. Pay for it.** Add 5 bps of cost and push the rehedges back up to 1260. The distribution is tight and centered on a loss. The continuous hedging the textbook asks for is the most expensive strategy available.

That last one is the trade-off the book keeps circling back to: hedge too rarely and you're carrying variance, hedge too often and you're paying the market for the privilege. There is no setting on that slider that returns the riskless position from the derivation.

## The Loop Itself

The widget hides the part I find most interesting as a programmer, which is how little code the strategy is. Simulate a price path, then walk it. At each step the shares I'm short gain or lose against the move, I look up the new delta, and I pay to trade the difference. At the end the option pays off.

```python
import random
from statistics import mean, stdev


def gbm_path(S0, vol, T, steps, rng):
    """Zero-drift geometric Brownian motion, steps + 1 prices."""
    h = T / steps
    path = [S0]
    for _ in range(steps):
        shock = rng.gauss(0, 1)
        path.append(path[-1] * exp(-0.5 * vol * vol * h + vol * sqrt(h) * shock))
    return path


def hedged_pnl(path, K, T, implied_vol, cost=0.0):
    """Long one call, short delta shares, rebalanced at every point on the path."""
    steps = len(path) - 1
    dt = T / steps
    premium, delta = bs_call(path[0], K, T, implied_vol)
    pnl = -premium - cost * delta * path[0]
    for i in range(1, steps + 1):
        S = path[i]
        pnl -= delta * (S - path[i - 1])          # the short shares
        new_delta = bs_call(S, K, T - i * dt, implied_vol)[1] if i < steps else 0.0
        pnl -= cost * abs(new_delta - delta) * S  # pay to rebalance
        delta = new_delta
    return pnl + max(path[-1] - K, 0.0)           # the option pays off
```

Notice the two volatilities live in different functions. `gbm_path` takes the volatility the world actually has; `hedged_pnl` takes the one I believe in. Nothing forces them to agree, and the hedger never gets to see the first one directly.

Running it a couple of thousand times per setting gives the same picture as the histogram above:

```python
def experiment(implied, realized, steps, cost=0.0, paths=2000, seed=1):
    rng = random.Random(seed)
    results = [
        hedged_pnl(gbm_path(100, realized, 0.25, steps, rng), 100, 0.25, implied, cost)
        for _ in range(paths)
    ]
    return mean(results), stdev(results)


for steps in (4, 16, 63, 252):
    m, s = experiment(implied=0.20, realized=0.20, steps=steps)
    print(f"rehedges={steps:4d}  mean={m:+.2f}  stdev={s:.2f}")
```

```text
rehedges=   4  mean=-0.03  stdev=1.60
rehedges=  16  mean=+0.00  stdev=0.81
rehedges=  63  mean=+0.00  stdev=0.42
rehedges= 252  mean=+0.00  stdev=0.21
```

Each row hedges four times as often as the one before and the standard deviation halves, which is the square-root rule from experiment 1. It's the same convergence rate as any Monte Carlo estimate, and for the same reason: I'm averaging independent errors.

Getting the volatility wrong, with 63 rehedges (once a trading day):

```text
realized=10%  mean=-1.98  stdev=0.55
realized=20%  mean=+0.00  stdev=0.42
realized=30%  mean=+1.98  stdev=1.11
```

Vega for this option is about 19.9 per unit of volatility, so being off by ten points should be worth about 1.99. The simulation lands on 1.98.

And with `cost=0.0005`, 5 bps on every trade:

```text
rehedges=   4  mean=-0.11  stdev=1.60
rehedges=  16  mean=-0.11  stdev=0.80
rehedges=  63  mean=-0.18  stdev=0.42
rehedges= 252  mean=-0.30  stdev=0.23
```

The standard deviation column barely changes from the free version. The mean column gets steadily worse. That's the whole trade-off in two columns of numbers: one is what I pay for certainty and the other is how much certainty I got.

## Past Delta

The simulator still cheats, since it holds volatility constant and never jumps. Most of the book lives in the territory past that:

- **Vega** is the exposure to implied volatility itself, and for a hedged book it's usually the one that actually matters.
- The Greeks aren't stable. Delta moves when volatility moves, vega moves when volatility moves, and delta drifts with nothing but the passage of time. Taleb has his own names for these (DdeltaDvol, DvegaDvol, delta bleed).
- **Shadow gamma** is his fix for the fact that volatility tends to change when the price does, so the gamma the model reports isn't the gamma you experience in a real move.

I'm not far enough along to write about those with any confidence, so I'll leave them for a later post.

## What a Programmer Takes From This

The Black-Scholes derivation is a spec written for a machine that doesn't exist: one with an infinitely fast clock, free operations and perfectly known inputs. The actual system runs a polling loop, the `for` loop in `hedged_pnl` above. It samples at intervals, each sample costs something, and one of its parameters is an estimate that turns out wrong in ways you only see afterwards.

Anyone who has run a control loop, a cache or a retry policy in production knows the shape of that problem. What I appreciate about the book is that it doesn't pretend the residue away; it gives each piece of it a name and a way to measure it. Understanding precisely how your model is wrong turns out to be most of the work.

## References

[1] Nassim Nicholas Taleb, [*Dynamic Hedging: Managing Vanilla and Exotic Options*](https://www.amazon.com/dp/0471152803), Wiley, 1997.

[2] Fischer Black and Myron Scholes, "The Pricing of Options and Corporate Liabilities," *Journal of Political Economy*, 1973.

[3] Emanuel Derman and Michael Kamal, "When You Cannot Hedge Continuously: The Corrections of Black-Scholes," *Risk*, 1999.

END OF LINE