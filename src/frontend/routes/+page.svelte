<script>
  let downloadImages = $state(false);
  let downloadAsPdf  = $state(false);
  let isPaidStory    = $state(false);
  let invalidUrl     = $state(false);
  let afterDownload  = $state(false);
  let loadingBook    = $state(false);
  let bookInfo       = $state(null);

  let credentials = $state({ username: '', password: '' });
  let downloadId  = $state('');
  /** @type {"story"|"part"|""} */
  let mode    = $state('');
  let inputUrl = $state('');

  /** @type {HTMLDialogElement} */
  let tutorialModal;

  let buttonDisabled = $derived(
    !inputUrl || (isPaidStory && !(credentials.username && credentials.password))
  );

  let dlUrl = $derived(
    `/download/` + downloadId +
    `?om=1` +
    (downloadImages ? '&download_images=true' : '') +
    (isPaidStory ? `&username=${encodeURIComponent(credentials.username)}&password=${encodeURIComponent(credentials.password)}` : '') +
    `&mode=${mode}` +
    (downloadAsPdf ? '&format=pdf' : '&format=epub')
  );

  /** @param {string} n */
  function fmt(n) {
    if (!n) return '0';
    if (n >= 1e6) return (n / 1e6).toFixed(1) + 'M';
    if (n >= 1e3) return (n / 1e3).toFixed(1) + 'K';
    return String(n);
  }

  /** @param {string} input */
  const setValid = (input) => { invalidUrl = false; inputUrl = input; downloadId = input; };
  /** @param {string} input */
  const setInvalid = (input) => { invalidUrl = true; inputUrl = input; downloadId = input; };

  /** @param {string} input */
  const setInputUrl = (input) => {
    input = input.toLowerCase();
    if (!input) { setValid(''); return; }
    if (/^\d+$/.test(input)) { mode = 'story'; setValid(input); return; }
    if (!input.includes('wattpad.com/')) { setInvalid(input.match(/\d+/g)?.join('') ?? ''); return; }
    if (input.includes('/story/')) {
      mode = 'story';
      setValid(input.split('-', 1)[0].split('?', 1)[0].split('/story/')[1]);
    } else if (input.includes('/stories/')) {
      mode = 'story';
      setValid(input.split('?', 1)[0].split('/stories/')[1]);
    } else {
      input = input.split('-', 1)[0].split('?', 1)[0].split('wattpad.com/')[1];
      if (/^\d+$/.test(input)) { mode = 'part'; setValid(input); }
      else setInvalid('');
    }
  };

  async function fetchBook() {
    if (!downloadId || invalidUrl) return;
    loadingBook = true;
    bookInfo = null;
    try {
      const storyId = mode === 'story' ? downloadId : null;
      if (!storyId) { loadingBook = false; return; }
      const res = await fetch(
        `https://www.wattpad.com/api/v3/stories/${storyId}?fields=id,title,cover,user(name),description,mainCategory,numParts,reads,votes`
      );
      if (!res.ok) throw new Error('not found');
      bookInfo = await res.json();
    } catch {
      bookInfo = null;
    } finally {
      loadingBook = false;
    }
  }

  function handleInput(e) {
    setInputUrl(e.target.value);
    bookInfo = null;
  }

  async function handleBlur() {
    if (downloadId && !invalidUrl && mode === 'story') await fetchBook();
  }

  async function handleKey(e) {
    if (e.key === 'Enter' && downloadId && !invalidUrl && mode === 'story') await fetchBook();
  }
</script>

<style>
  /* ─── Page shell ─── */
  .page {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 3rem 1.25rem 5rem;
    gap: 1.25rem;
  }

  /* ─── Header ─── */
  .header { text-align: center; margin-bottom: 0.5rem; }
  .brand {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 8px;
  }
  .brand-icon {
    width: 36px; height: 36px;
    background: var(--text);
    border-radius: 9px;
    display: flex; align-items: center; justify-content: center;
    flex-shrink: 0;
  }
  .brand-icon svg { width: 18px; height: 18px; fill: var(--bg); }
  .brand-name {
    font-family: 'Instrument Serif', serif;
    font-size: 22px;
    color: var(--text);
    letter-spacing: -0.2px;
  }
  .tagline { font-size: 13px; color: var(--muted); font-weight: 300; }

  /* ─── Card ─── */
  .card {
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    padding: 1.5rem;
    width: 100%;
    max-width: 540px;
  }

  /* ─── Form elements ─── */
  .field-label {
    display: block;
    font-size: 11px;
    font-weight: 500;
    color: var(--hint);
    letter-spacing: 0.7px;
    text-transform: uppercase;
    margin-bottom: 7px;
  }
  .input-row { display: flex; gap: 8px; }
  .url-input {
    flex: 1;
    border: 1px solid var(--border-md);
    border-radius: var(--radius-sm);
    padding: 10px 13px;
    font-size: 14px;
    font-family: 'DM Sans', sans-serif;
    color: var(--text);
    background: var(--bg-input);
    outline: none;
    transition: border-color 0.15s, background 0.15s;
  }
  .url-input:focus { border-color: var(--text); background: var(--bg-card); }
  .url-input::placeholder { color: var(--hint); }
  .url-input.invalid { border-color: var(--red); }

  .btn-primary {
    background: var(--accent);
    color: var(--bg);
    border: none;
    border-radius: var(--radius-sm);
    padding: 10px 18px;
    font-size: 13px;
    font-weight: 500;
    font-family: 'DM Sans', sans-serif;
    cursor: pointer;
    transition: opacity 0.15s;
    white-space: nowrap;
    display: flex;
    align-items: center;
    gap: 7px;
  }
  .btn-primary:hover:not(:disabled) { opacity: 0.8; }
  .btn-primary:disabled { opacity: 0.4; cursor: not-allowed; }

  .btn-ghost {
    background: transparent;
    border: 1px solid var(--border-md);
    border-radius: var(--radius-sm);
    padding: 9px 16px;
    font-size: 12px;
    font-family: 'DM Sans', sans-serif;
    color: var(--muted);
    cursor: pointer;
    transition: border-color 0.15s, color 0.15s;
  }
  .btn-ghost:hover { border-color: var(--text); color: var(--text); }

  /* ─── Helper text ─── */
  .hint-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-top: 7px;
    min-height: 18px;
  }
  .err-txt { font-size: 12px; color: var(--red); display: flex; align-items: center; gap: 5px; }
  .err-txt svg { width: 13px; height: 13px; flex-shrink: 0; }
  .help-btn {
    font-size: 12px;
    color: var(--muted);
    background: none; border: none;
    cursor: pointer; padding: 0;
    text-decoration: underline;
    text-underline-offset: 3px;
  }
  .help-btn:hover { color: var(--text); }

  /* ─── Divider ─── */
  .divider {
    border: none;
    border-top: 1px solid var(--border);
    margin: 1.1rem 0;
  }

  /* ─── Toggles ─── */
  .toggle-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 4px 0;
  }
  .toggle-label { font-size: 13px; color: var(--text); display: flex; flex-direction: column; gap: 1px; }
  .toggle-sub { font-size: 11px; color: var(--muted); font-weight: 300; }
  .toggle {
    position: relative;
    width: 36px; height: 20px;
    flex-shrink: 0;
  }
  .toggle input { opacity: 0; width: 0; height: 0; }
  .slider {
    position: absolute;
    inset: 0;
    background: var(--border-md);
    border-radius: 999px;
    cursor: pointer;
    transition: background 0.2s;
  }
  .slider::before {
    content: '';
    position: absolute;
    width: 14px; height: 14px;
    left: 3px; top: 3px;
    background: white;
    border-radius: 50%;
    transition: transform 0.2s;
  }
  .toggle input:checked + .slider { background: var(--text); }
  .toggle input:checked + .slider::before { transform: translateX(16px); }

  /* ─── Paid fields ─── */
  .paid-fields { display: flex; flex-direction: column; gap: 8px; margin-top: 10px; }
  .text-input {
    width: 100%;
    border: 1px solid var(--border-md);
    border-radius: var(--radius-sm);
    padding: 9px 13px;
    font-size: 14px;
    font-family: 'DM Sans', sans-serif;
    color: var(--text);
    background: var(--bg-input);
    outline: none;
    transition: border-color 0.15s;
  }
  .text-input:focus { border-color: var(--text); }
  .text-input::placeholder { color: var(--hint); }

  /* ─── Download btn ─── */
  .dl-row { margin-top: 1.1rem; }
  .btn-dl {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    width: 100%;
    padding: 12px;
    background: var(--accent);
    color: var(--bg);
    border: none;
    border-radius: var(--radius-sm);
    font-size: 14px;
    font-weight: 500;
    font-family: 'DM Sans', sans-serif;
    cursor: pointer;
    text-decoration: none;
    transition: opacity 0.15s;
  }
  .btn-dl:hover:not(.disabled) { opacity: 0.8; }
  .btn-dl.disabled { opacity: 0.4; pointer-events: none; }
  .btn-dl.pdf { background: var(--green); }

  /* ─── Book preview card ─── */
  .book-card {
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    width: 100%;
    max-width: 540px;
    overflow: hidden;
    animation: fadeUp 0.22s ease both;
  }
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(8px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .book-inner {
    display: flex;
    gap: 18px;
    padding: 20px;
    align-items: flex-start;
  }
  .cover {
    width: 88px;
    height: 132px;
    border-radius: 7px;
    object-fit: cover;
    flex-shrink: 0;
    border: 1px solid var(--border);
    display: block;
  }
  .cover-ph {
    width: 88px;
    height: 132px;
    border-radius: 7px;
    background: var(--bg-input);
    border: 1px solid var(--border);
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .cover-ph svg { width: 22px; height: 22px; fill: var(--hint); }
  .book-meta { flex: 1; min-width: 0; }
  .book-cat {
    font-size: 10px;
    font-weight: 500;
    color: var(--hint);
    text-transform: uppercase;
    letter-spacing: 0.7px;
    margin-bottom: 4px;
  }
  .book-title {
    font-family: 'Instrument Serif', serif;
    font-size: 18px;
    color: var(--text);
    line-height: 1.25;
    margin-bottom: 3px;
    word-break: break-word;
  }
  .book-author { font-size: 12px; color: var(--muted); margin-bottom: 10px; }
  .book-author b { color: var(--text); font-weight: 500; }
  .book-desc {
    font-size: 12px;
    color: var(--muted);
    line-height: 1.55;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
    margin-bottom: 12px;
    font-weight: 300;
  }
  .book-stats { display: flex; gap: 14px; }
  .stat-val { font-size: 13px; font-weight: 500; color: var(--text); }
  .stat-lbl { font-size: 10px; color: var(--hint); }

  /* ─── Book loading skeleton ─── */
  .skeleton-card {
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    width: 100%;
    max-width: 540px;
    padding: 20px;
    display: flex;
    gap: 18px;
  }
  .skel {
    background: var(--border);
    border-radius: 5px;
    animation: pulse 1.4s ease infinite;
  }
  @keyframes pulse { 0%,100% { opacity: 1; } 50% { opacity: 0.45; } }
  .skel-cover { width: 88px; height: 132px; border-radius: 7px; flex-shrink: 0; }
  .skel-lines { flex: 1; display: flex; flex-direction: column; gap: 8px; padding-top: 4px; }
  .skel-line { height: 12px; }

  /* ─── After download ─── */
  .after-wrap {
    text-align: center;
    max-width: 440px;
    padding: 2rem;
    background: var(--bg-card);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
  }
  .after-icon {
    width: 48px; height: 48px;
    background: var(--bg-input);
    border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    margin: 0 auto 1rem;
  }
  .after-icon svg { width: 22px; height: 22px; fill: var(--green); }
  .after-title {
    font-family: 'Instrument Serif', serif;
    font-size: 22px;
    color: var(--text);
    margin-bottom: 0.5rem;
  }
  .after-sub { font-size: 13px; color: var(--muted); font-weight: 300; margin-bottom: 1.5rem; line-height: 1.6; }
  .btn-more {
    display: inline-flex;
    align-items: center;
    gap: 7px;
    padding: 11px 22px;
    border: 1px solid var(--border-md);
    border-radius: var(--radius-sm);
    font-size: 13px;
    font-family: 'DM Sans', sans-serif;
    color: var(--text);
    background: transparent;
    cursor: pointer;
    transition: border-color 0.15s;
    text-decoration: none;
  }
  .btn-more:hover { border-color: var(--text); }

  /* ─── Spinner ─── */
  .spin {
    width: 14px; height: 14px;
    border: 2px solid rgba(255,255,255,0.3);
    border-top-color: #fff;
    border-radius: 50%;
    animation: rot 0.7s linear infinite;
    display: inline-block;
  }
  @keyframes rot { to { transform: rotate(360deg); } }

  /* ─── Footer ─── */
  .footer {
    position: fixed;
    bottom: 0; left: 0; right: 0;
    padding: 10px 16px;
    text-align: center;
    font-size: 11px;
    color: var(--hint);
    font-style: italic;
    font-weight: 300;
    letter-spacing: 0.03em;
    pointer-events: none;
  }

  /* ─── Modal ─── */
  dialog {
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    background: var(--bg-card);
    color: var(--text);
    padding: 0;
    max-width: 420px;
    width: 90%;
    box-shadow: 0 16px 48px rgba(0,0,0,0.15);
  }
  dialog::backdrop { background: rgba(0,0,0,0.4); }
  .modal-inner { padding: 1.5rem; }
  .modal-title { font-family: 'Instrument Serif', serif; font-size: 18px; margin-bottom: 1rem; }
  .modal-close {
    float: right;
    background: none; border: none;
    font-size: 18px; cursor: pointer;
    color: var(--muted); line-height: 1;
    margin-top: -2px;
  }
  .modal-close:hover { color: var(--text); }
  .steps { list-style: decimal; padding-left: 1.2rem; display: flex; flex-direction: column; gap: 10px; }
  .steps li { font-size: 13px; color: var(--muted); line-height: 1.55; }
  .mono {
    font-family: monospace;
    font-size: 12px;
    background: var(--bg-input);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 2px 6px;
  }
  .hl { background: #fef3c7; border-radius: 3px; padding: 1px 4px; color: #92400e; }

  @media (max-width: 480px) {
    .book-inner { flex-direction: column; }
    .cover, .cover-ph { width: 80px; height: 120px; }
    .skel-cover { width: 80px; height: 120px; }
  }
</style>

<div class="page">

  {#if !afterDownload}

    <!-- Header -->
    <div class="header">
      <div class="brand">
        <div class="brand-icon">
          <svg viewBox="0 0 24 24"><path d="M18 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2zM6 4h5v8l-2.5-1.5L6 12V4z"/></svg>
        </div>
        <span class="brand-name">WP Downloader</span>
      </div>
      <div class="tagline">Cole o link do Wattpad e baixe como EPUB ou PDF</div>
    </div>

    <!-- Search card -->
    <div class="card">
      <label class="field-label" for="wp-url">Link do livro</label>
      <div class="input-row">
        <input
          id="wp-url"
          class="url-input"
          class:invalid={invalidUrl}
          type="text"
          placeholder="https://www.wattpad.com/story/..."
          bind:value={() => inputUrl, setInputUrl}
          oninput={handleInput}
          onblur={handleBlur}
          onkeydown={handleKey}
        />
      </div>

      <div class="hint-row">
        {#if invalidUrl}
          <span class="err-txt">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><circle cx="12" cy="16" r="0.5" fill="currentColor"/>
            </svg>
            URL inválida — veja
            <button class="help-btn" onclick={() => tutorialModal.showModal()}>como obter o link</button>
          </span>
        {:else}
          <button class="help-btn" onclick={() => tutorialModal.showModal()}>Como obter o link?</button>
        {/if}
      </div>

      <hr class="divider" />

      <!-- Toggles -->
      <div style="display:flex;flex-direction:column;gap:10px;">
        <div class="toggle-row">
          <span class="toggle-label">
            Incluir imagens
            <span class="toggle-sub">Download mais lento</span>
          </span>
          <label class="toggle">
            <input type="checkbox" bind:checked={downloadImages} />
            <span class="slider"></span>
          </label>
        </div>

        <div class="toggle-row">
          <span class="toggle-label">
            Formato PDF
            <span class="toggle-sub">Padrão: EPUB</span>
          </span>
          <label class="toggle">
            <input type="checkbox" bind:checked={downloadAsPdf} />
            <span class="slider"></span>
          </label>
        </div>

        <div class="toggle-row">
          <span class="toggle-label">
            História paga (comprada)
          </span>
          <label class="toggle">
            <input type="checkbox" bind:checked={isPaidStory} />
            <span class="slider"></span>
          </label>
        </div>
      </div>

      {#if isPaidStory}
        <div class="paid-fields">
          <input class="text-input" type="text"     placeholder="Usuário Wattpad"  bind:value={credentials.username} />
          <input class="text-input" type="password" placeholder="Senha"            bind:value={credentials.password} />
        </div>
      {/if}

      <!-- Download button -->
      <div class="dl-row">
        <a
          class="btn-dl"
          class:pdf={downloadAsPdf}
          class:disabled={buttonDisabled}
          href={dlUrl}
          onclick={() => { if (!buttonDisabled) afterDownload = true; }}
        >
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
            <path d="M12 15V3m0 12l-4-4m4 4l4-4"/>
            <path d="M2 17l.621 2.485A2 2 0 0 0 4.561 21h14.878a2 2 0 0 0 1.94-1.515L22 17"/>
          </svg>
          Baixar {downloadAsPdf ? 'PDF' : 'EPUB'}
        </a>
      </div>
    </div>

    <!-- Book preview skeleton -->
    {#if loadingBook}
      <div class="skeleton-card">
        <div class="skel skel-cover"></div>
        <div class="skel-lines">
          <div class="skel skel-line" style="width:55%"></div>
          <div class="skel skel-line" style="width:75%"></div>
          <div class="skel skel-line" style="width:40%"></div>
          <div class="skel skel-line" style="width:90%;margin-top:6px"></div>
          <div class="skel skel-line" style="width:80%"></div>
          <div class="skel skel-line" style="width:60%"></div>
        </div>
      </div>
    {/if}

    <!-- Book preview -->
    {#if bookInfo && !loadingBook}
      <div class="book-card">
        <div class="book-inner">
          {#if bookInfo.cover}
            <img class="cover" src={bookInfo.cover} alt="Capa de {bookInfo.title}" />
          {:else}
            <div class="cover-ph">
              <svg viewBox="0 0 24 24"><path d="M18 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2zM6 4h5v8l-2.5-1.5L6 12V4z"/></svg>
            </div>
          {/if}
          <div class="book-meta">
            {#if bookInfo.mainCategory}
              <div class="book-cat">{bookInfo.mainCategory}</div>
            {/if}
            <div class="book-title">{bookInfo.title}</div>
            <div class="book-author">por <b>{bookInfo.user?.name || 'Desconhecido'}</b></div>
            {#if bookInfo.description}
              <div class="book-desc">{bookInfo.description}</div>
            {/if}
            <div class="book-stats">
              <div>
                <div class="stat-val">{fmt(bookInfo.reads)}</div>
                <div class="stat-lbl">leituras</div>
              </div>
              <div>
                <div class="stat-val">{fmt(bookInfo.votes)}</div>
                <div class="stat-lbl">votos</div>
              </div>
              <div>
                <div class="stat-val">{bookInfo.numParts ?? '—'}</div>
                <div class="stat-lbl">partes</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    {/if}

  {:else}

    <!-- After download -->
    <div class="after-wrap">
      <div class="after-icon">
        <svg viewBox="0 0 24 24"><path d="M20 6L9 17l-5-5"/></svg>
      </div>
      <div class="after-title">Download iniciado!</div>
      <p class="after-sub">
        Seu arquivo está sendo gerado e será baixado em breve.<br/>
        Aguarde alguns instantes.
      </p>
      <button
        class="btn-more"
        onclick={() => { afterDownload = false; inputUrl = ''; bookInfo = null; }}
      >
        ← Baixar outro livro
      </button>
    </div>

  {/if}

  <!-- Fixed footer -->
  <div class="footer">by Sylva.jpp</div>

</div>

<!-- Tutorial modal -->
<dialog bind:this={tutorialModal}>
  <div class="modal-inner">
    <button class="modal-close" onclick={() => tutorialModal.close()}>✕</button>
    <div class="modal-title">Como obter o link</div>
    <ol class="steps">
      <li>Abra o livro no site ou app do Wattpad e copie a URL.</li>
      <li>
        Exemplo válido:
        <span class="mono">wattpad.com/<span class="hl">story</span>/237369078-titulo</span>
      </li>
      <li>
        Ou também:
        <span class="mono">https://www.wattpad.com/939103774-titulo</span>
      </li>
      <li>Cole o link no campo e aguarde o preview do livro aparecer.</li>
      <li>Clique em <b>Baixar EPUB</b> ou <b>Baixar PDF</b>.</li>
    </ol>
  </div>
</dialog>
