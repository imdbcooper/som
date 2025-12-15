---
{"dg-publish":true,"permalink":"/published/blog/blog/","title":"Блог | Мои статьи и туториалы"}
---


# 📝 Блог

Здесь я делюсь знаниями и опытом через статьи, туториалы и размышления.

---

<style>
  .blog-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
    margin: 30px 0;
  }

  .blog-card {
    background: white;
    border: 1px solid #e0e0e0;
    border-radius: 12px;
    overflow: hidden;
    transition: all 0.3s ease;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    text-decoration: none;
    color: inherit;
    cursor: pointer;
  }

  .blog-card:hover {
    transform: translateY(-8px);
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.15);
  }

  .blog-card-image {
    width: 100%;
    height: 200px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 48px;
  }

  .blog-card-content {
    padding: 20px;
  }

  .blog-card-title {
    font-size: 18px;
    font-weight: 600;
    margin: 0 0 10px 0;
    color: #333;
  }

  .blog-card-excerpt {
    font-size: 14px;
    color: #666;
    margin: 0 0 15px 0;
    line-height: 1.5;
  }

  .blog-card-meta {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 12px;
    color: #999;
  }

  .blog-card-tag {
    display: inline-block;
    background: #f0f0f0;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 11px;
    color: #667eea;
    font-weight: 500;
  }
</style>

<div id="blog-container" class="blog-grid"></div>

<script>
  // Конфигурация — адаптируйте под вашу структуру!
  const BLOG_FOLDER = 'Blog';
  const PUBLISHED_ONLY = true;

  // Эта функция будет вызвана Digital Garden при загрузке
  async function loadBlogPosts() {
    try {
      // Получаем список файлов из папки Blog
      const response = await fetch(`/${BLOG_FOLDER}`);
      const html = await response.text();
      
      // Парсим HTML и извлекаем ссылки на файлы
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');
      const links = Array.from(doc.querySelectorAll('a'))
        .filter(a => a.href.endsWith('.md'))
        .map(a => a.textContent);

      // Для каждого файла загружаем его содержимое
      const posts = await Promise.all(
        links.map(async (filename) => {
          const fileResponse = await fetch(`/${BLOG_FOLDER}/${filename}`);
          const content = await fileResponse.text();
          return parseBlogPost(filename, content);
        })
      );

      // Сортируем по дате (новые первыми)
      posts.sort((a, b) => new Date(b.date) - new Date(a.date));

      // Рендерим карточки
      renderBlogCards(posts);
    } catch (error) {
      console.error('Ошибка при загрузке статей:', error);
      document.getElementById('blog-container').innerHTML = 
        '<p style="color: red;">Ошибка при загрузке статей</p>';
    }
  }

  function parseBlogPost(filename, content) {
    // Извлекаем метаданные из YAML
    const metaMatch = content.match(/^---([\s\S]*?)---/);
    const meta = {};
    
    if (metaMatch) {
      const lines = metaMatch.trim().split('\n');
      lines.forEach(line => {
        const [key, ...valueParts] = line.split(':');
        const value = valueParts.join(':').trim().replace(/["']/g, '');
        meta[key.trim()] = value;
      });
    }

    return {
      title: meta.title || filename.replace('.md', ''),
      date: meta.date || new Date().toISOString().split('T'),
      excerpt: meta.excerpt || 'Нет описания',
      category: meta.category || 'Статья',
      emoji: meta.emoji || '📝',
      readTime: meta.readTime || '5',
      filename: filename,
      published: meta['dg-publish'] === 'true'
    };
  }

  function renderBlogCards(posts) {
    const container = document.getElementById('blog-container');
    
    // Фильтруем только опубликованные
    const publishedPosts = posts.filter(p => p.published || !PUBLISHED_ONLY);

    if (publishedPosts.length === 0) {
      container.innerHTML = '<p>Статьи ещё не опубликованы</p>';
      return;
    }

    container.innerHTML = publishedPosts.map(post => `
      <a href="../${BLOG_FOLDER}/${post.filename.replace('.md', '')}" class="blog-card">
        <div class="blog-card-image">${post.emoji}</div>
        <div class="blog-card-content">
          <h3 class="blog-card-title">${post.title}</h3>
          <p class="blog-card-excerpt">${post.excerpt}</p>
          <div class="blog-card-meta">
            <span>${formatDate(post.date)}</span>
            <span class="blog-card-tag">${post.category}</span>
          </div>
        </div>
      </a>
    `).join('');
  }

  function formatDate(dateStr) {
    const date = new Date(dateStr);
    return date.toLocaleDateString('ru-RU', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  }

  // Запускаем при загрузке страницы
  loadBlogPosts();
</script>