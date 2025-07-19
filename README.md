## 👑 Creator ( Joker Exodus )
©2025 - Joker

c'est ma base de bot simple réalisée par moi même et non modifier 🧚

<p align="center">
  <img src="https://files.catbox.moe/i04w1g.jpeg" width="800"/>
</p>

## Developer ( Joker Exodus )
©2025 - Joker

Ceci est ma base de bot WhatsApp simple, alors n'hésitez pas à l'utiliser si vous êtes intéressé.
const { default: makeWASocket, useMultiFileAuthState, DisconnectReason, makeInMemoryStore } = require('@whiskeysockets/baileys');
const { Boom } = require('@hapi/boom');
const P = require('pino');
const fs = require('fs');

const startSock = async () => {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info');
    const sock = makeWASocket({
        logger: P({ level: 'silent' }),
        printQRInTerminal: true,
        auth: state
    });

    sock.ev.on('creds.update', saveCreds);

    sock.ev.on('messages.upsert', async ({ messages, type }) => {
        const msg = messages[0];
        if (!msg.message || msg.key.fromMe) return;
        const from = msg.key.remoteJid;
        const body = msg.message.conversation || msg.message.extendedTextMessage?.text || "";

        const reply = (text) => sock.sendMessage(from, { text }, { quoted: msg });

        if (body.startsWith('!menu')) {
            reply('*Commandes disponibles :*
!menu
!tagall
!kick
!purge');
        }

        if (body.startsWith('!tagall') && msg.key.participant) {
            const groupMetadata = await sock.groupMetadata(from);
            const participants = groupMetadata.participants.map(p => p.id);
            const mentions = participants.map(p => p.replace(/@s.whatsapp.net/, ''));
            await sock.sendMessage(from, {
                text: `@${mentions.join(' @')}`,
                mentions: participants
            });
        }

        if (body.startsWith('!kick')) {
            if (msg.key.participant) {
                const number = body.split(' ')[1];
                if (!number) return reply('Utilisation : !kick 123456789');
                await sock.groupParticipantsUpdate(from, [`${number}@s.whatsapp.net`], 'remove');
            }
        }

        if (body.startsWith('!purge')) {
            reply('Commande purge non implémentée. (fonction test)');
        }
    });

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        if (connection === 'close') {
            const shouldReconnect = (lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut;
            if (shouldReconnect) {
                startSock();
            }
        }
    });

    return sock;
}
npm install
node index.js
startSock();npm install
node index.js
