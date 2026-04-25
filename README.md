# blog-auto const express = require("express");
const axios = require("axios");
const fs = require("fs");
const slugify = require("slugify");

const app = express();
const PORT = process.env.PORT || 3000;

// 🔑 COLOQUE SUA CHAVE
const OPENAI_KEY = process.env.OPENAI_KEY || "SUA_CHAVE_AQUI";

// banco simples
const DB = "posts.json";
if (!fs.existsSync(DB)) fs.writeFileSync(DB, "[]");

function getPosts() {
  return JSON.parse(fs.readFileSync(DB));
}

function savePosts(posts) {
  fs.writeFileSync(DB, JSON.stringify(posts, null, 2));
}

// 🔥 TEMAS QUE VENDEM
const temas = [
  "melhor celular até 1500 2026",
  "melhor notebook custo beneficio",
  "fone bluetooth bom e barato",
  "cadeira gamer vale a pena",
  "produto mais vendido amazon vale a pena",
  "curso hotmart que realmente funciona"
];

// 🤖 IA
async function gerarPost(tema) {
  try {
    const res = await axios.post(
      "https://api.openai.com/v1/chat/completions",
      {
        model: "gpt-4o-mini",
        messages: [{
          role: "user",
          content: `
Crie artigo SEO persuasivo:
- Título forte
- H2 H3
- Benefícios
- Comparação
- CTA de compra
Tema: ${tema}
`
        }]
      },
      {
        headers: { Authorization: `Bearer ${OPENAI_KEY}` }
      }
    );

    return res.data.choices[0].message.content;

  } catch (e) {
    return null;
  }
}

// 🚀 GERAR POSTS
async function gerarPosts(qtd = 5) {
  const posts = getPosts();

  for (let i = 0; i < qtd; i++) {
    const tema = temas[Math.floor(Math.random() * temas.length)];
    const slug = slugify(tema + "-" + Date.now() + i, { lower: true });

    const conteudo = await gerarPost(tema);

    if (conteudo) {
      posts.push({ titulo: tema, slug, conteudo });
      console.log("Post:", slug);
    }
  }

  savePosts(posts);
}

// ⏰ AUTOMÁTICO
setInterval(() => {
  console.log("Gerando automático...");
  gerarPosts(3);
}, 1000 * 60 * 60 * 4);

// 🌐 HOME
app.get("/", (req, res) => {
  const posts = getPosts().reverse();

  let html = `
  <html>
  <head>
    <title>Melhores Produtos 2026</title>
  </head>
  <body>

  <h1>🔥 Melhores Produtos</h1>

  <ul>
  `;

  posts.forEach(p => {
    html += `<li><a href="/post/${p.slug}">${p.titulo}</a></li>`;
  });

  html += `
  </ul>
  </body>
  </html>
  `;

  res.send(html);
});

// 📄 POST
app.get("/post/:slug", (req, res) => {
  const post = getPosts().find(p => p.slug === req.params.slug);

  if (!post) return res.send("Não encontrado");

  res.send(`
  <html>
  <body>

  <h1>${post.titulo}</h1>
  <p>${post.conteudo}</p>

  <h2>🔥 Melhor oferta</h2>

  <a href="https://www.amazon.com.br" target="_blank">
  👉 VER PREÇO AGORA
  </a>

  <br><br>
  <a href="/">Voltar</a>

  </body>
  </html>
  `);
});

// 🚀 START
app.listen(PORT, async () => {
  console.log("Rodando...");
  await gerarPosts(10);
});
