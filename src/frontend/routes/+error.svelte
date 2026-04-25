<script>
  import { page } from '$app/state';
</script>

<style>
  .wrap {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    text-align: center;
    position: relative;
    overflow: hidden;
  }
  .blob {
    position: absolute;
    top: -80px;
    left: 50%;
    transform: translateX(-50%);
    width: 400px;
    height: 400px;
    border-radius: 50%;
    background: var(--border);
    opacity: 0.4;
    z-index: 0;
    pointer-events: none;
  }
  .content {
    position: relative;
    z-index: 1;
    max-width: 420px;
  }
  .code {
    font-family: 'Instrument Serif', serif;
    font-size: 88px;
    line-height: 1;
    letter-spacing: -3px;
    color: var(--text);
    animation: up 0.4s ease both;
  }
  .line {
    width: 36px;
    height: 2px;
    background: var(--text);
    margin: 1.1rem auto;
    animation: up 0.4s 0.08s ease both;
    opacity: 0;
  }
  .title {
    font-family: 'Instrument Serif', serif;
    font-size: 20px;
    color: var(--text);
    margin-bottom: 0.4rem;
    animation: up 0.4s 0.13s ease both;
    opacity: 0;
  }
  .sub {
    font-size: 14px;
    color: var(--muted);
    font-weight: 300;
    margin-bottom: 2rem;
    animation: up 0.4s 0.18s ease both;
    opacity: 0;
  }
  .btn {
    display: inline-block;
    font-family: 'DM Sans', sans-serif;
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.07em;
    text-transform: uppercase;
    text-decoration: none;
    padding: 11px 28px;
    border: 1.5px solid var(--text);
    color: var(--text);
    background: transparent;
    border-radius: 3px;
    cursor: pointer;
    transition: background 0.18s, color 0.18s;
    animation: up 0.4s 0.24s ease both;
    opacity: 0;
  }
  .btn:hover { background: var(--text); color: var(--bg); }
  .credit {
    position: absolute;
    bottom: 1.2rem;
    left: 0; right: 0;
    text-align: center;
    font-size: 11px;
    font-style: italic;
    font-weight: 300;
    color: var(--hint);
    letter-spacing: 0.04em;
    z-index: 2;
  }
  @keyframes up {
    from { opacity: 0; transform: translateY(12px); }
    to   { opacity: 1; transform: translateY(0); }
  }
</style>

<div class="wrap">
  <div class="blob"></div>
  <div class="content">
    <div class="code">{page.status || 'Erro'}</div>
    <div class="line"></div>
    <div class="title">Algo deu errado</div>
    <p class="sub">{page.error?.message || 'Não foi possível processar sua solicitação.'}</p>
    <a href="/" class="btn">← Voltar ao início</a>
  </div>
  <div class="credit">by Sylva.jpp</div>
</div>
