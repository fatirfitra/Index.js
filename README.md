const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys');
const fs = require('fs');

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState('session');

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    });

    sock.ev.on('creds.update', saveCreds);

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        if(connection === 'open') console.log('✅ Bot nyambung!');
    });

    sock.ev.on('messages.upsert', async m => {
        const msg = m.messages[0];
        if (!msg.message || msg.key.fromMe) return;

        const text = msg.message.conversation || msg.message.extendedTextMessage?.text;
        const sender = msg.key.remoteJid;

        console.log('Pesan dari', sender, ':', text);

        // Command simpel
        if (text === '.ping') {
            await sock.sendMessage(sender, { text: 'pong 🏓' });
        }
        if (text === '.menu') {
            await sock.sendMessage(sender, { text: 'Menu:\n.ping - tes bot\n.menu - liat menu' });
        }
    });
}

startBot();
