---
layout: default
---

<section class="profile-card">
  <img class="profile-photo" src="{{ site.logo | relative_url }}" alt="Kelvin Li">
  <div class="profile-info">
    <h1>Kelvin Li</h1>
    <p class="profile-bio">Engineering @ <a href="https://www.mcmaster.ca/">McMaster University</a>.</p>
    <div class="profile-social">
      <a href="https://www.kaggle.com/likelvin" aria-label="Kaggle"><i class="fas fa-user-graduate"></i></a>
      <a href="https://github.com/li-kelvin" aria-label="GitHub"><i class="fab fa-github"></i></a>
      <a href="https://li-kelvin.github.io/blog/" aria-label="Blog"><i class="fas fa-feather-alt"></i></a>
      <a href="https://github.com/li-kelvin?tab=repositories" aria-label="Project Repository"><i class="far fa-bookmark"></i></a>
      <a href="Resume.pdf" aria-label="Resume"><i class="far fa-file"></i></a>
    </div>
  </div>
</section>

## About
<p>I'm a competitive, data-driven problem solver (he/him) who likes turning insights into real-world impact. Most of my work sits at the intersection of technical support, data, and internal tooling — resolving complex issues, documenting repeatable processes, and helping teams deliver faster, more consistent support. That's meant SQL-driven root cause analysis, cross-functional client launches, and working closely with Support, Ops, and Customer Success across both MSP and SaaS environments. Outside of work, I put the same discipline into coaching 300+ athletes — the best teams, on and off the court, run on communication, resilience, and a system that scales.</p>

## Journal
<p class="section-desc">Every day, colored by how it went.</p>

<div class="heatmap-card">
  <div class="heatmap-scroll">
    <div class="heatmap-inner">
      <div id="heatmap-months" class="heatmap-months"></div>
      <div class="heatmap-row">
        <div id="heatmap-daylabels" class="heatmap-daylabels"></div>
        <div id="heatmap-grid" class="heatmap-grid"></div>
      </div>
    </div>
  </div>
  <div class="heatmap-footer">
    <span class="heatmap-legend-label">Rough</span>
    <div class="heatmap-legend">
      <span class="heatmap-cell tier-0"></span>
      <span class="heatmap-cell tier-1"></span>
      <span class="heatmap-cell tier-2"></span>
      <span class="heatmap-cell tier-3"></span>
      <span class="heatmap-cell tier-4"></span>
    </div>
    <span class="heatmap-legend-label">Great</span>
  </div>
</div>

<div id="journal-debrief-overlay" class="journal-debrief-overlay" hidden>
  <div class="journal-debrief-modal">
    <button type="button" id="journal-debrief-close" class="journal-debrief-close" aria-label="Close">
      <i class="fas fa-times"></i>
    </button>
    <h3 id="journal-debrief-date" class="journal-debrief-date"></h3>

    <div id="journal-debrief-pin-gate" class="journal-lock">
      <i class="fas fa-lock"></i>
      <p class="journal-lock-text">Enter your PIN to view this day's debrief.</p>
      <form id="journal-debrief-pin-form" class="journal-pin-form">
        <input type="password" inputmode="numeric" pattern="[0-9]*" maxlength="4" id="journal-debrief-pin-input" placeholder="PIN" aria-label="PIN">
        <button type="submit">Unlock</button>
      </form>
      <p id="journal-debrief-pin-error" class="journal-pin-error" hidden>Wrong PIN.</p>
    </div>

    <div id="journal-debrief-content" class="journal-list" hidden></div>
  </div>
</div>

### Earlier Writing
<div class="row-list">
  <a class="row-list-item" href="https://li-kelvin.github.io/blog/posts/12-01-2021/">
    <span class="row-title">What's the point of living?</span>
    <span class="row-meta">12/01/2021 <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
  <a class="row-list-item" href="https://li-kelvin.github.io/blog/posts/11-30-2021/">
    <span class="row-title">I'm wasting my life.</span>
    <span class="row-meta">11/30/2021 <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
</div>

<script>
(function () {
  var PIN_HASH = "72f93cdffdd25a0cc5ed855d2388671b7e715ca09a8c844a90cb3e4e7398fc45";
  var journalRaw = {{ site.journal | jsonify }};
  var dayEntries = {};
  journalRaw.forEach(function (e) {
    if (typeof e.speaking_score !== 'number' || typeof e.engagement_score !== 'number') return;
    var key = String(e.date).slice(0, 10);
    if (!dayEntries[key]) dayEntries[key] = [];
    dayEntries[key].push(e);
  });

  var sortedDayKeys = Object.keys(dayEntries).sort();

  function avgFor(entries) {
    var sum = entries.reduce(function (a, e) { return a + (e.speaking_score + e.engagement_score) / 2; }, 0);
    return sum / entries.length;
  }

  function dayRatingFor(entries) {
    if (!entries) return null;
    var found = entries.find(function (e) { return typeof e.day_rating === 'number'; });
    return found ? found.day_rating : null;
  }

  function dayNumberFor(key) {
    return sortedDayKeys.indexOf(key) + 1;
  }

  function orderedEntries(entries) {
    return (entries || []).slice().sort(function (a, b) {
      if (a.period === b.period) return 0;
      return a.period === 'morning' ? -1 : 1;
    });
  }

  function ordinal(n) {
    var s = ['th', 'st', 'nd', 'rd'], v = n % 100;
    return n + (s[(v - 20) % 10] || s[v] || s[0]);
  }

  function formatEntrySubtitle(e) {
    var dateSource = e.display_date || e.date;
    var d = new Date(String(dateSource).slice(0, 10) + 'T00:00:00');
    var weekday = d.toLocaleDateString('en-US', { weekday: 'short' });
    var month = d.toLocaleDateString('en-US', { month: 'short' });
    var label = e.period === 'morning' ? '🌅 Morning' : '🌙 Evening';
    var dateStr = weekday + ' ' + month + ' ' + ordinal(d.getDate());
    return label + ' - ' + dateStr + (e.time ? ' @' + e.time : '');
  }

  function tierFor(entries) {
    if (!entries || !entries.length) return 0;
    var rating = dayRatingFor(entries);
    var score = rating !== null ? rating * 10 : avgFor(entries);
    if (score < 50) return 1;
    if (score < 65) return 2;
    if (score < 80) return 3;
    return 4;
  }

  function pad(n) { return n < 10 ? '0' + n : '' + n; }
  function toKey(d) { return d.getFullYear() + '-' + pad(d.getMonth() + 1) + '-' + pad(d.getDate()); }

  var monthsEl = document.getElementById('heatmap-months');
  var gridEl = document.getElementById('heatmap-grid');
  var labelsEl = document.getElementById('heatmap-daylabels');

  ['', 'Mon', '', 'Wed', '', 'Fri', ''].forEach(function (label) {
    var span = document.createElement('span');
    span.textContent = label;
    labelsEl.appendChild(span);
  });

  var today = new Date();
  var start = new Date(today);
  start.setDate(start.getDate() - 371);
  start.setDate(start.getDate() - start.getDay());

  var monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
  var lastMonth = null;
  var weekIndex = 0;
  var cursor = new Date(start);

  while (cursor <= today) {
    if (cursor.getDay() === 0) {
      if (cursor.getMonth() !== lastMonth) {
        var label = document.createElement('span');
        label.textContent = monthNames[cursor.getMonth()];
        label.style.gridColumnStart = weekIndex + 1;
        monthsEl.appendChild(label);
        lastMonth = cursor.getMonth();
      }
      weekIndex++;
    }

    var key = toKey(cursor);
    var entries = cursor > today ? null : dayEntries[key];
    var tier = cursor > today ? -1 : tierFor(entries);
    var cell = document.createElement('span');
    cell.className = 'heatmap-cell tier-' + tier;
    cell.dataset.key = key;
    var ratingForTitle = entries ? dayRatingFor(entries) : null;
    cell.title = cursor.toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' }) +
      (entries
        ? (ratingForTitle !== null ? ' — ' + ratingForTitle + '/10' : ' — day avg ' + Math.round(avgFor(entries)) + '/100')
        : ' — no entry');
    if (entries) {
      cell.tabIndex = 0;
      cell.setAttribute('role', 'button');
      cell.style.cursor = 'pointer';
    }
    gridEl.appendChild(cell);

    cursor.setDate(cursor.getDate() + 1);
  }

  // Click-to-open debrief, PIN-gated
  var overlay = document.getElementById('journal-debrief-overlay');
  var dateEl = document.getElementById('journal-debrief-date');
  var pinGate = document.getElementById('journal-debrief-pin-gate');
  var pinForm = document.getElementById('journal-debrief-pin-form');
  var pinInput = document.getElementById('journal-debrief-pin-input');
  var pinError = document.getElementById('journal-debrief-pin-error');
  var contentEl = document.getElementById('journal-debrief-content');
  var closeBtn = document.getElementById('journal-debrief-close');
  var activeKey = null;

  function isUnlocked() {
    try { return sessionStorage.getItem('journal-unlocked') === '1'; } catch (e) { return false; }
  }

  function sha256Hex(text) {
    var data = new TextEncoder().encode(text);
    return crypto.subtle.digest('SHA-256', data).then(function (buf) {
      return Array.prototype.map.call(new Uint8Array(buf), function (b) {
        return b.toString(16).padStart(2, '0');
      }).join('');
    });
  }

  function scoreTier(score) {
    if (score >= 80) return 'score-good';
    if (score >= 60) return 'score-mid';
    return 'score-low';
  }

  function renderDebrief(key) {
    var entries = orderedEntries(dayEntries[key]);
    var rating = dayRatingFor(entries);
    dateEl.textContent = 'Day ' + dayNumberFor(key) + (rating !== null ? ' · ' + rating + '/10' : '');
    contentEl.innerHTML = '';
    entries.forEach(function (e) {
      var entry = document.createElement('div');
      entry.className = 'journal-entry';
      entry.innerHTML =
        '<div class="journal-entry-top">' +
          '<span class="journal-entry-period"></span>' +
        '</div>' +
        '<p class="journal-entry-notes"></p>' +
        '<div class="journal-scores">' +
          '<span class="score-chip ' + scoreTier(e.speaking_score) + '">Speaking <strong>' + e.speaking_score + '</strong>/100</span>' +
          '<span class="score-chip ' + scoreTier(e.engagement_score) + '">Engagement <strong>' + e.engagement_score + '</strong>/100</span>' +
        '</div>';
      entry.querySelector('.journal-entry-period').textContent = formatEntrySubtitle(e);
      entry.querySelector('.journal-entry-notes').textContent = e.notes || '';
      contentEl.appendChild(entry);
    });
  }

  function openDebrief(key) {
    activeKey = key;
    overlay.hidden = false;
    if (isUnlocked()) {
      pinGate.hidden = true;
      contentEl.hidden = false;
      renderDebrief(key);
    } else {
      pinGate.hidden = false;
      contentEl.hidden = true;
      pinError.hidden = true;
      pinInput.value = '';
      pinInput.focus();
    }
  }

  function closeDebrief() {
    overlay.hidden = true;
    activeKey = null;
  }

  gridEl.addEventListener('click', function (e) {
    var cell = e.target.closest('.heatmap-cell');
    if (cell && cell.dataset.key && dayEntries[cell.dataset.key]) openDebrief(cell.dataset.key);
  });

  gridEl.addEventListener('keydown', function (e) {
    if (e.key !== 'Enter' && e.key !== ' ') return;
    var cell = e.target.closest('.heatmap-cell');
    if (cell && cell.dataset.key && dayEntries[cell.dataset.key]) {
      e.preventDefault();
      openDebrief(cell.dataset.key);
    }
  });

  closeBtn.addEventListener('click', closeDebrief);
  overlay.addEventListener('click', function (e) {
    if (e.target === overlay) closeDebrief();
  });
  document.addEventListener('keydown', function (e) {
    if (e.key === 'Escape' && !overlay.hidden) closeDebrief();
  });

  pinForm.addEventListener('submit', function (e) {
    e.preventDefault();
    sha256Hex(pinInput.value).then(function (hash) {
      if (hash === PIN_HASH) {
        try { sessionStorage.setItem('journal-unlocked', '1'); } catch (err) {}
        pinGate.hidden = true;
        contentEl.hidden = false;
        if (activeKey) renderDebrief(activeKey);
      } else {
        pinError.hidden = false;
        pinInput.value = '';
        pinInput.focus();
      }
    });
  });
})();
</script>

## Projects
<div class="row-list">
  <a class="row-list-item" href="https://www.kaggle.com/likelvin/shot-selection">
    <span class="row-title">Kobe Bryant Shot Visualizations</span>
    <span class="row-meta">Kaggle <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
  <a class="row-list-item" href="https://www.kaggle.com/likelvin/nba-salary-prediction-w-regression-model">
    <span class="row-title">NBA Salary Predictor</span>
    <span class="row-meta">Kaggle <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
  <a class="row-list-item" href="https://github.com/li-kelvin/arduino-sumo-robot">
    <span class="row-title">Arduino Sumo Bot</span>
    <span class="row-meta">GitHub <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
</div>

## Reading
<p class="section-desc">Papers and books worth remembering.</p>

### Papers
<div class="row-list">
  <a class="row-list-item" href="https://blog.acolyer.org/">
    <span class="row-title">The Morning Paper</span>
    <span class="row-meta"><i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
  <a class="row-list-item" href="https://www.gsd.inesc-id.pt/~ler/conferencedates.html">
    <span class="row-title">Most Academic Paper</span>
    <span class="row-meta"><i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
</div>

### Books
<div class="row-list">
  <a class="row-list-item" href="https://li-kelvin.github.io/blog/posts/the-subtle-art-of-not-giving-a-fuck/">
    <span class="row-title">The Subtle Art Of Not Giving A F*ck</span>
    <span class="row-meta">Mark Manson <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
  <a class="row-list-item" href="https://li-kelvin.github.io/blog/posts/the-communist-manifesto/">
    <span class="row-title">The Communist Manifesto</span>
    <span class="row-meta">Marx &amp; Engels <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
  <a class="row-list-item" href="https://li-kelvin.github.io/blog/posts/animal-farm/">
    <span class="row-title">Animal Farm</span>
    <span class="row-meta">George Orwell <i class="fas fa-chevron-right row-chevron"></i></span>
  </a>
</div>

## Thanks
<div class="thanks-emoji">🙏</div>
<p class="thanks-intro">Over the years, I have met many people that left a positive impact on me.<br>Thanks for stumbling into my life.<br><br>I tried to summarize one learning from each.</p>
<div class="row-list">
  <div class="row-list-item">
    <span class="row-title">Mr. Coleman</span>
    <span class="row-meta">if someone punches you, punch them back</span>
  </div>
  <div class="row-list-item">
    <span class="row-title">Oliver Li</span>
    <span class="row-meta">getting work done &gt; other stuff</span>
  </div>
  <div class="row-list-item">
    <span class="row-title">Jeffery Jiang</span>
    <span class="row-meta">great humans are still just humans</span>
  </div>
  <div class="row-list-item">
    <span class="row-title">The Old People on their morning walk</span>
    <span class="row-meta">smiles change lives</span>
  </div>
</div>
