# Baileys Modifikasi

Versi modifikasi dari library Baileys untuk WhatsApp Web API dengan fitur tambahan dan penyempurnaan.

[![npm version](https://badge.fury.io/js/baileys-modified.svg)](https://badge.fury.io/js/baileys-modified)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js Version](https://img.shields.io/badge/node-%3E%3D16.0.0-brightgreen.svg)](https://nodejs.org/)
[![Downloads](https://img.shields.io/npm/dm/baileys-modified.svg)](https://www.npmjs.com/package/baileys-modified)

## 📋 Daftar Isi

- [Fitur](#fitur)
- [Instalasi](##instalasi)
- [Penggunaan Dasar](#penggunaan-dasar)
- [Konfigurasi](#konfigurasi)
- [API Reference](#api-reference)
- [Contoh Penggunaan](#contoh-penggunaan)
- [FAQ](#faq)
- [Kontribusi](#kontribusi)
- [Lisensi](#lisensi)

## 🚀 Fitur

### Fitur Utama
- ✅ Koneksi stabil ke WhatsApp Web
- ✅ Multi-device support
- ✅ Mengirim pesan teks, media, dan dokumen
- ✅ Menerima dan menangani pesan masuk
- ✅ Group management (buat, kelola member, dll)
- ✅ Status/Story management

### Fitur Modifikasi
- 🔥 **Auto reconnect** - Reconnect otomatis saat koneksi terputus
- 🔥 **Message queue** - Antrian pesan untuk pengiriman yang lebih stabil
- 🔥 **Rate limiting** - Pembatasan kecepatan untuk menghindari spam detection
- 🔥 **Enhanced logging** - Sistem logging yang lebih detail
- 🔥 **Session management** - Pengelolaan sesi yang lebih baik
- 🔥 **Error handling** - Penanganan error yang lebih robust

## 📦 Instalasi

### Persyaratan
- Node.js v16 atau lebih baru
- NPM atau Yarn

### Install via NPM
```bash
npm install Baileys
```

### Install via Yarn
```bash
yarn add Baileys
```

### Install dari Source
```bash
git clone https://github.com/fajar-reiva-cahya/Baileys.git
cd Baileys
npm install
npm run build
```

## 🛠 Penggunaan Dasar

### Inisialisasi Bot
```javascript
const { default: makeWASocket, DisconnectReason, useMultiFileAuthState } = require('Baileys')
const { Boom } = require('@hapi/boom')

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')
    
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update
        if(connection === 'close') {
            const shouldReconnect = (lastDisconnect.error instanceof Boom)?.output?.statusCode !== DisconnectReason.loggedOut
            console.log('connection closed due to ', lastDisconnect.error, ', reconnecting ', shouldReconnect)
            if(shouldReconnect) {
                startBot()
            }
        } else if(connection === 'open') {
            console.log('opened connection')
        }
    })

    sock.ev.on('creds.update', saveCreds)
    
    return sock
}

startBot()
```

### Mengirim Pesan
```javascript
// Kirim pesan teks
await sock.sendMessage('628123456789@s.whatsapp.net', { text: 'Hello World!' })

// Kirim gambar
await sock.sendMessage('628123456789@s.whatsapp.net', { 
    image: { url: 'https://example.com/image.jpg' },
    caption: 'Ini adalah gambar'
})

// Kirim dokumen
await sock.sendMessage('628123456789@s.whatsapp.net', {
    document: { url: 'https://example.com/document.pdf' },
    mimetype: 'application/pdf',
    fileName: 'document.pdf'
})
```

## ⚙️ Konfigurasi

### Konfigurasi Dasar
```javascript
const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true,
    // Konfigurasi modifikasi
    connectTimeoutMs: 60000,
    defaultQueryTimeoutMs: 60000,
    keepAliveIntervalMs: 10000,
    // Auto reconnect
    shouldReconnect: true,
    maxReconnectAttempts: 5,
    reconnectDelay: 3000,
    // Rate limiting
    messageRateLimit: {
        maxMessages: 20,
        windowMs: 60000
    }
})
```

### Konfigurasi Logging
```javascript
const pino = require('pino')

const sock = makeWASocket({
    auth: state,
    logger: pino({ 
        level: 'debug',
        transport: {
            target: 'pino-pretty',
            options: {
                colorize: true
            }
        }
    })
})
```

## 📚 API Reference

### Events

#### `connection.update`
Dipanggil ketika status koneksi berubah.
```javascript
sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect, qr } = update
    // Handle connection updates
})
```

#### `messages.upsert`
Dipanggil ketika pesan baru diterima.
```javascript
sock.ev.on('messages.upsert', (messageUpdate) => {
    const { messages, type } = messageUpdate
    // Handle incoming messages
})
```

#### `groups.update`
Dipanggil ketika informasi grup diperbarui.
```javascript
sock.ev.on('groups.update', (groupUpdates) => {
    // Handle group updates
})
```

### Methods

#### `sendMessage(jid, content, options)`
Mengirim pesan ke nomor atau grup tertentu.

**Parameters:**
- `jid` (string): ID WhatsApp (format: number@s.whatsapp.net)
- `content` (object): Konten pesan
- `options` (object): Opsi tambahan

#### `groupCreate(subject, participants)`
Membuat grup baru.

**Parameters:**
- `subject` (string): Nama grup
- `participants` (array): Array nomor peserta

## 💡 Contoh Penggunaan

### Bot Sederhana
```javascript
const { default: makeWASocket, useMultiFileAuthState } = require('Baileys')

async function createBot() {
    const { state, saveCreds } = await useMultiFileAuthState('session')
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })

    sock.ev.on('messages.upsert', async (messageUpdate) => {
        const { messages } = messageUpdate
        const message = messages[0]
        
        if (!message.key.fromMe && message.message?.conversation) {
            const text = message.message.conversation
            const sender = message.key.remoteJid
            
            // Auto reply
            if (text.toLowerCase() === 'ping') {
                await sock.sendMessage(sender, { text: 'Pong!' })
            }
        }
    })

    sock.ev.on('creds.update', saveCreds)
}

createBot()
```

### Bot dengan Command Handler
```javascript
const commands = {
    ping: {
        execute: async (sock, message) => {
            await sock.sendMessage(message.key.remoteJid, { text: 'Pong! 🏓' })
        }
    },
    info: {
        execute: async (sock, message) => {
            const info = `
📱 *Bot Information*
- Name: Baileys Modified Bot
- Version: 1.0.0
- Status: Online ✅
            `
            await sock.sendMessage(message.key.remoteJid, { text: info })
        }
    }
}

sock.ev.on('messages.upsert', async (messageUpdate) => {
    const { messages } = messageUpdate
    const message = messages[0]
    
    if (!message.key.fromMe && message.message?.conversation) {
        const text = message.message.conversation.trim()
        const args = text.split(' ')
        const command = args[0].toLowerCase().replace('!', '')
        
        if (commands[command]) {
            try {
                await commands[command].execute(sock, message)
            } catch (error) {
                console.error('Command error:', error)
            }
        }
    }
})
```

## ❓ FAQ

### Q: Bagaimana cara mengatasi QR code yang tidak muncul?
A: Pastikan `printQRInTerminal: true` di konfigurasi dan terminal mendukung output QR code.

### Q: Bot sering disconnect, bagaimana solusinya?
A: Gunakan fitur auto reconnect dengan mengatur `shouldReconnect: true` di konfigurasi.

### Q: Bagaimana cara mengirim pesan ke grup?
A: Gunakan group ID sebagai `jid`, format: `groupId@g.us`

### Q: Apakah bot ini aman untuk production?
A: Ya, tapi pastikan untuk menggunakan proper error handling dan rate limiting.

## 🤝 Kontribusi

Kontribusi sangat diterima! Silakan:

1. Fork repository ini
2. Buat branch feature (`git checkout -b feature/AmazingFeature`)
3. Commit perubahan (`git commit -m 'Add some AmazingFeature'`)
4. Push ke branch (`git push origin feature/AmazingFeature`)
5. Buat Pull Request

### Guidelines
- Ikuti style code yang ada
- Tambahkan tests untuk fitur baru
- Update dokumentasi jika diperlukan
- Pastikan semua tests passing

## 📄 Lisensi

Distributed under the MIT License. See `LICENSE` for more information.

## 🙏 Acknowledgments

- [Baileys](https://github.com/WhiskeySockets/Baileys) - Library WhatsApp Web API original
- [WhatsApp Web](https://web.whatsapp.com/) - Platform WhatsApp Web
- Semua kontributor yang telah membantu pengembangan project ini

## 📞 Support

Jika mengalami masalah atau memiliki pertanyaan:

- Buat [Issue](https://github.com/fajar-reiva-cahya/Baileys/issues) di GitHub
- Contact [Telegram](https://t.me/siraybukandeveloper)

---

⭐ Jangan lupa star repository ini jika bermanfaat!
