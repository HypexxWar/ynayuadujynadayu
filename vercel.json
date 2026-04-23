const express = require('express');
const bodyParser = require('body-parser');
const sqlite3 = require('sqlite3').verbose();
const axios = require('axios');
const QRCode = require('qrcode');
const app = express();

// CONFIG
const WEBHOOK = "https://discord.com/api/webhooks/1496674523033108510/t8xV3CUE_57XfA5oni9dXADe__ibLckyTFnUTdomu5BJufpdCcjN2EzZC0nJ1vnpr91n";
const CHAVE_PIX = "THEKRIAMC@PIX.COM";

// DATABASE
const db = new sqlite3.Database('./database.db');

// CRIA TABELA SE NÃO EXISTIR
db.run(`
CREATE TABLE IF NOT EXISTS orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nickname TEXT,
    product TEXT,
    price INTEGER,
    status TEXT
)
`);

app.set('view engine', 'ejs');
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static('public'));

// ==========================
// FUNÇÃO PIX
// ==========================
function gerarPayloadPix(nome, chave, valor) {
    const valorFormatado = valor.toFixed(2);
    return `00020126580014BR.GOV.BCB.PIX0136${chave}52040000530398654${valorFormatado.length}${valorFormatado}5802BR5920${nome}6009SAO PAULO62070503***6304`;
}

// ==========================
// LOJA
// ==========================
app.get('/', (req, res) => {
    res.render('index');
});

// ==========================
// GERAR PIX
// ==========================
app.post('/gerar-pix', async (req, res) => {
    const { nickname, product } = req.body;

    let price = 0;
    if (product === "VIP Ouro") price = 10;
    else if (product === "VIP Prata") price = 5;
    else if (product === "Kit PvP") price = 7;

    console.log("Gerando PIX:", nickname, product, price);

    const payload = gerarPayloadPix("KriaMC", CHAVE_PIX, price);
    const qrCode = await QRCode.toDataURL(payload);

    res.render('pix', {
        nickname,
        product,
        price,
        qrCode,
        payload
    });
});

// ==========================
// CONFIRMAR PEDIDO
// ==========================
app.post('/confirmar-pagamento', (req, res) => {
    const { nickname, product, price } = req.body;

    console.log("Salvando pedido:", nickname, product, price);

    db.run(
        "INSERT INTO orders (nickname, product, price, status) VALUES (?, ?, ?, ?)",
        [nickname, product, price, 'PENDENTE'],
        function (err) {
            if (err) {
                console.error("Erro DB:", err);
                return res.send("Erro ao salvar pedido");
            }

            console.log("Pedido salvo ID:", this.lastID);

            // ENVIA PRO DISCORD
            axios.post(WEBHOOK, {
                content: `🛒 NOVO PEDIDO\n\n👤 Nick: ${nickname}\n📦 Produto: ${product}\n`
            })
            .then(r => {
                console.log("Discord OK:", r.status);
            })
            .catch(err => {
                console.error("Erro Discord:", err.response ? err.response.data : err.message);
            });

            res.send("Pedido enviado! Aguarde confirmação.");
        }
    );
});

// ==========================
// LISTAR PEDIDOS (ADMIN)
// ==========================
app.get('/panel', (req, res) => {
    db.all("SELECT * FROM orders", [], (err, rows) => {
        res.render('panel', { orders: rows });
    });
});

// ==========================
// ATUALIZAR STATUS + DISCORD
// ==========================
app.get('/update/:id/:status', (req, res) => {
    const { id, status } = req.params;

    db.run("UPDATE orders SET status=? WHERE id=?", [status, id], (err) => {
        if (err) return res.send("Erro");

        db.get("SELECT * FROM orders WHERE id=?", [id], (err, pedido) => {

            axios.post(WEBHOOK, {
                content: `📦 PEDIDO ATUALIZADO\n\n🆔 ID: ${pedido.id}\n👤 ${pedido.nickname}\n📦 ${pedido.product}\n💰 R$${pedido.price}\n📌 ${pedido.status}`
            })
            .then(r => console.log("Discord update OK"))
            .catch(err => console.error("Erro Discord:", err.message));

            res.redirect('/panel');
        });
    });
});

// ==========================
app.listen(3000, () => {
    console.log("Servidor rodando em http://localhost:3000");
});
