<script>
  let url = $state('');
  let loading = $state(false);
  let error = $state('');
  let book = $state(null);
  let downloaded = $state(false);

  function extractStoryId(input) {
    input = input.trim();
    if (/^\d+$/.test(input)) return input;
    const match = input.match(/\/(\d{6,})/);
    return match ? match[1] : null;
  }

  async function fetchBookInfo() {
    error = '';
    book = null;
    downloaded = false;
    const storyId = extractStoryId(url);
    if (!storyId) {
      error = 'URL inválida. Cole um link do Wattpad.';
      return;
    }
    loading = true;
    try {
      const res = await fetch(
        `https://www.wattpad.com/api/v3/stories/${storyId}?fields=id,title,cover,user(name),description,mainCategory,numParts,reads,votes,language`
      );
      if (!res.ok) throw new Error('Livro não encontrado.');
      book = await res.json();
    } catch (e) {
      error = e.message || 'Erro ao buscar informações do livro.';
    } finally {
      loading = false;
    }
  }

  async function downloadBook() {
    if (!book) return;
    loading = true;
    error = '';
    try {
      const res = await fetch(`/download/${book.id}?mode=story`);
      if (!res.ok) throw new Error('Falha ao gerar o arquivo.');
      const blob = await res.blob();
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `${book.title}.epub`;
      a.click();
      downloaded = true;
    } catch (e) {
      error = e.message || 'Erro ao baixar o livro.';
    } finally {
      loading = false;
    }
  }

  function handleKey(e) {
    if (e.key === 'Enter') fetchBookInfo();
  }

  function reset() {
    url = '';
    book = null;
    error = '';
    downloaded = false;
  }

  function formatNumber(n) {
    if (!n) return '0';
    if (n >= 1_000_000) return (n / 1_000_000).toFixed(1) + 'M';
    if (n >= 1_000) return (n / 1_000).toFixed(1) + 'K';
    return n.toString();
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;1,400&family=DM+Sans:wght@300;400;500&display=swap');

  :global(body) {
    margin: 0;
    background: #f7f4ef;
    min-height: 100vh;
    font-family: 'DM Sans', sans-serif;
  }

  .page {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 48px 20px 80px;
    background: #f7f4ef;
  }

  .header { text-align: center; margin-bottom: 48px; }

  .logo-mark {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 16px;
  }

  .logo-icon {
    width: 36px;
    height: 36px;
    background: #1a1a1a;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .logo-icon svg { width: 20px; height: 20px; fill: #f7f4ef; }

  .logo-text {
    font-family: 'Lora', serif;
    font-size: 22px;
    font-weight: 600;
    color: #1a1a1a;
    letter-spacing: -0.3px;
  }

  .tagline { font-size: 14px; color: #888; font-weight: 300; letter-spacing: 0.3px; }

  .search-card {
    background: #fff;
    border-radius: 16px;
    padding: 32px;
    width: 100%;
    max-width: 560px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 16px rgba(0,0,0,0.06);
    border: 1px solid rgba(0,0,0,0.06);
  }

  .search-label {
    display: block;
    font-size: 12px;
    font-weight: 500;
    color: #aaa;
    letter-spacing: 0.8px;
    text-transform: uppercase;
    margin-bottom: 10px;
  }

  .input-row { display: flex; gap: 10px; }

  .url-input {
    flex: 1;
    border: 1.5px solid #e5e0d8;
    border-radius: 10px;
    padding: 12px 16px;
    font-size: 14px;
    font-family: 'DM Sans', sans-serif;
    color: #1a1a1a;
    background: #faf9f7;
    outline: none;
    transition: border-color 0.15s;
  }

  .url-input:focus { border-color: #1a1a1a; background: #fff; }
  .url-input::placeholder { color: #bbb; }

  .btn-search {
    background: #1a1a1a;
    color: #fff;
    border: none;
    border-radius: 10px;
    padding: 12px 20px;
    font-size: 14px;
    font-weight: 500;
    font-family: 'DM Sans', sans-serif;
    cursor: pointer;
    transition: opacity 0.15s, transform 0.1s;
    white-space: nowrap;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .btn-search:hover:not(:disabled) { opacity: 0.85; }
  .btn-search:active:not(:disabled) { transform: scale(0.98); }
  .btn-search:disabled { opacity: 0.5; cursor: not-allowed; }

  .error-msg {
    margin-top: 14px;
    font-size: 13px;
    color: #c0392b;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .book-card {
    background: #fff;
    border-radius: 16px;
    width: 100%;
    max-width: 560px;
    margin-top: 20px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 16px rgba(0,0,0,0.06);
    border: 1px solid rgba(0,0,0,0.06);
    overflow: hidden;
    animation: fadeUp 0.25s ease both;
  }

  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(10px); }
    to   { opacity: 1; transform: translateY(0); }
  }

  .book-inner { display: flex; gap: 24px; padding: 28px; }
  .cover-wrap { flex-shrink: 0; width: 100px; }

  .cover-img {
    width: 100px;
    height: 150px;
    object-fit: cover;
    border-radius: 8px;
    display: block;
    box-shadow: 0 4px 16px rgba(0,0,0,0.15);
  }

  .cover-placeholder {
    width: 100px;
    height: 150px;
    border-radius: 8px;
    background: #e8e4dd;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .cover-placeholder svg { width: 32px; height: 32px; fill: #bbb; }

  .book-info { flex: 1; min-width: 0; display: flex; flex-direction: column; }

  .book-category {
    font-size: 11px;
    font-weight: 500;
    color: #888;
    letter-spacing: 0.8px;
    text-transform: uppercase;
    margin-bottom: 6px;
  }

  .book-title {
    font-family: 'Lora', serif;
    font-size: 20px;
    font-weight: 600;
    color: #1a1a1a;
    line-height: 1.3;
    margin: 0 0 4px;
    word-break: break-word;
  }

  .book-author { font-size: 13px; color: #888; margin-bottom: 14px; }
  .book-author span { color: #555; }

  .book-desc {
    font-size: 13px;
    color: #666;
    line-height: 1.6;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
    margin-bottom: 16px;
    flex: 1;
  }

  .book-stats { display: flex; gap: 16px; }
  .stat { display: flex; flex-direction: column; align-items: flex-start; }
  .stat-val { font-size: 14px; font-weight: 500; color: #1a1a1a; }
  .stat-lbl { font-size: 11px; color: #aaa; }

  .dl-bar {
    border-top: 1px solid #f0ece5;
    padding: 20px 28px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
  }

  .btn-dl {
    background: #1a1a1a;
    color: #fff;
    border: none;
    border-radius: 10px;
    padding: 11px 22px;
    font-size: 14px;
    font-weight: 500;
    font-family: 'DM Sans', sans-serif;
    cursor: pointer;
    transition: opacity 0.15s, transform 0.1s, background 0.2s;
    display: flex;
    align-items: center;
    gap: 8px;
    flex-shrink: 0;
  }

  .btn-dl:hover:not(:disabled) { opacity: 0.85; }
  .btn-dl:active:not(:disabled) { transform: scale(0.98); }
  .btn-dl:disabled { opacity: 0.45; cursor: not-allowed; }
  .btn-dl.success { background: #2d7a4f; }

  .btn-reset {
    background: transparent;
    border: 1.5px solid #e5e0d8;
    border-radius: 10px;
    padding: 10px 18px;
    font-size: 13px;
    font-family: 'DM Sans', sans-serif;
    color: #888;
    cursor: pointer;
    transition: border-color 0.15s, color 0.15s;
  }

  .btn-reset:hover { border-color: #1a1a1a; color: #1a1a1a; }

  .spinner {
    width: 16px;
    height: 16px;
    border: 2px solid rgba(255,255,255,0.3);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    flex-shrink: 0;
  }

  @keyframes spin { to { transform: rotate(360deg); } }

  .footer { margin-top: 48px; font-size: 12px; color: #bbb; text-align: center; }
  .footer a { color: #bbb; text-decoration: none; }
  .footer a:hover { color: #888; }

  @media (max-width: 480px) {
    .book-inner { flex-direction: column; align-items: center; text-align: center; }
    .book-stats { justify-content: center; }
    .book-author { text-align: center; }
    .dl-bar { flex-direction: column; }
  }
</style>

<div class="page">
  <div class="header">
    <div class="logo-mark">
      <div class="logo-icon">
        <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path d="M18 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2zM6 4h5v8l-2.5-1.5L6 12V4z"/>
        </svg>
      </div>
      <span class="logo-text">WP Downloader</span>
    </div>
    <div class="tagline">Cole o link de qualquer livro do Wattpad e baixe como EPUB</div>
  </div>

  <div class="search-card">
    <label class="search-label" for="url-input">Link do livro</label>
    <div class="input-row">
      <input
        id="url-input"
        class="url-input"
        type="text"
        bind:value={url}
        onkeydown={handleKey}
        placeholder="https://www.wattpad.com/story/..."
        disabled={loading}
      />
      <button
        class="btn-search"
        onclick={fetchBookInfo}
        disabled={loading || !url.trim()}
      >
        {#if loading && !book}
          <span class="spinner"></span>
          Buscando
        {:else}
          Buscar
        {/if}
      </button>
    </div>

    {#if error}
      <div class="error-msg">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"/>
          <line x1="12" y1="8" x2="12" y2="12"/>
          <line x1="12" y1="16" x2="12.01" y2="16"/>
        </svg>
        {error}
      </div>
    {/if}
  </div>

  {#if book}
    <div class="book-card">
      <div class="book-inner">
        <div class="cover-wrap">
          {#if book.cover}
            <img class="cover-img" src={book.cover} alt="Capa de {book.title}" />
          {:else}
            <div class="cover-placeholder">
              <svg viewBox="0 0 24 24">
                <path d="M18 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2zM6 4h5v8l-2.5-1.5L6 12V4z"/>
              </svg>
            </div>
          {/if}
        </div>
        <div class="book-info">
          {#if book.mainCategory}
            <div class="book-category">{book.mainCategory}</div>
          {/if}
          <h1 class="book-title">{book.title}</h1>
          <div class="book-author">por <span>{book.user?.name || 'Desconhecido'}</span></div>
          {#if book.description}
            <p class="book-desc">{book.description}</p>
          {/if}
          <div class="book-stats">
            <div class="stat">
              <span class="stat-val">{formatNumber(book.reads)}</span>
              <span class="stat-lbl">leituras</span>
            </div>
            <div class="stat">
              <span class="stat-val">{formatNumber(book.votes)}</span>
              <span class="stat-lbl">votos</span>
            </div>
            <div class="stat">
              <span class="stat-val">{book.numParts ?? '—'}</span>
              <span class="stat-lbl">partes</span>
            </div>
          </div>
        </div>
      </div>

      <div class="dl-bar">
        <button class="btn-reset" onclick={reset}>Trocar livro</button>
        <button
          class="btn-dl {downloaded ? 'success' : ''}"
          onclick={downloadBook}
          disabled={loading}
        >
          {#if loading}
            <span class="spinner"></span>
            Gerando…
          {:else if downloaded}
            ✓ Baixado
          {:else}
            ↓ Baixar EPUB
          {/if}
        </button>
      </div>
    </div>
  {/if}

  <div class="footer">
    by <a href="https://github.com/Sylva-jpp" target="_blank">Sylva.jpp</a>
    &nbsp;·&nbsp; API em <a href="/docs" target="_blank">/docs</a>
  </div>
</div>
