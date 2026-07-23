---
name: bridge
description: The Telegram bridge — a #! resident long-polling the Bot API with $TG_TOKEN off the keyring; incoming messages ride as its voice (mail to wearers, pictures and sound seen in the letter itself, every file landed on the ground), the send tool answers into the chat with words, a file, or both. Raises the chat's own "typing…" while the machine works. Dormant until a kit wears it. Not a wire tool (no schema on purpose) — the body is a server for the spawn door.
---
#!/usr/bin/env node
// telegram/bridge — one wire between this machine and a Telegram bot.
//
// The shape is the house resident idiom (boot/file.md): newline JSON-RPC
// over stdio, tools/list + tools/call, notifications for the world's own
// words. The TOKEN rides the env by NAME ($TG_TOKEN — the vault's law:
// the value never appears in this text or any call), the API base is a
// SETTING ({{TG_API}} renders from the volume's settings record at
// every stand), and the chat is LEARNED from the first incoming message
// — long-polling, so no public door and no webhook is ever needed.
//
// A chat carries THREE planes, and each lands on a surface the machine
// already owns:
//   WORDS — the message text rides the voice as a letter that wakes the
//     companion.
//   FILES — what the user attaches LANDS ON THE GROUND (~/telegram/in,
//     one directory per file so a second "photo.jpg" never eats the
//     first) and the letter names the absolute path: the ground is the
//     one home every call and every service shares, so the file is
//     already where the machine's hands work — no download door, no
//     ferrying. Pictures and sound ALSO ride the letter itself as MCP
//     content blocks (boot/resident-skill's notification dialect): the
//     machine lands those bytes on its blob plane and the companion
//     SEES the picture instead of reading its filename. The way out is
//     the same one tool — send(file:) reads a path off the ground and
//     uploads it as the kind its BYTES are (a picture as a picture, a
//     clip that plays, anything else a document), because which Bot API
//     endpoint carries which bytes is physics, never a choice to hand a
//     model.
//   THE WAIT — a chat that shows nothing looks broken, so the bridge
//     raises Telegram's own "typing…" the instant a message lands (the
//     machine HAS it, long before the first token is drawn), refreshes
//     it while the work runs, wears the matching upload action while a
//     file goes up, and rests it when the answer is out. Physics, not a
//     knob: nothing for the companion to call, nothing to remember.
'use strict';
const fs = require('node:fs');
const path = require('node:path');

const TOKEN = process.env.TG_TOKEN || '';
const API_RAW = '{{TG_API}}';
// an unrendered placeholder (a bare adoption outside an install) falls
// to the public API — the render is the install's gift, not a demand
const API = API_RAW.startsWith('{{') || API_RAW === '' ? 'https://api.telegram.org' : API_RAW;
const HOME = process.env.HOME || process.cwd();
// the learned chat survives a restart on the ground — the send target
// must not die with the process (the wear re-stands it without notice)
const CHAT_FILE = path.join(HOME, '.telegram-chat');
// where an incoming file lands: under the GROUND, so every path this
// bridge names is one the machine's file hands and exec calls already
// reach at that exact spelling
const IN_DIR = path.join(HOME, 'telegram', 'in');

// Bytes that ride the LETTER itself (base64, the MCP content dialect) —
// file.md's own wall, the same number: past it the landed path is the
// whole answer and the companion reads the file with its hands.
const INLINE_MAX = 5 * 1024 * 1024;
// What a model may be SHOWN inline: the raster types every provider
// dialect is known to take. Anything else travels as its path alone.
const SEEABLE = new Set(['image/jpeg', 'image/png', 'image/gif', 'image/webp']);
// Telegram's own walls — the caller must not have to know them.
const TEXT_MAX = 4096; // one message's text; a longer answer rides as several
const CAPTION_MAX = 1024; // a caption riding a file; longer words follow as their own message
const PHOTO_MAX = 10 * 1024 * 1024; // past it Telegram takes an image as a document only
const UPLOAD_MAX = 50 * 1024 * 1024; // past it the Bot API takes nothing at all
// The chat action holds for ~5 seconds on Telegram's side, so a longer
// wait is REFRESHED; and no chat types forever — a companion that never
// answers (a dead wearer, a refusal) must not leave a phantom at work.
const TYPING_EVERY = 4_000;
const TYPING_MAX = 5 * 60_000;

const send = m => process.stdout.write(`${JSON.stringify(m)}\n`);
const ok = (id, result) => send({ jsonrpc: '2.0', id, result });
const told = text => ({ content: [{ type: 'text', text }], isError: false });
const refused = text => ({ content: [{ type: 'text', text }], isError: true });
// A server does not SPEAK before the wire is dressed: a notification
// that outruns the tools/list answer wakes a wearer whose kit cannot
// expand yet — the send tool is not on its wire, and the reply dies
// on "no such operation" (observed live: a Telegram backlog arrives
// in the FIRST poll, microseconds before the handshake settles). The
// voice queues until the roster has been served, then flushes.
let dressed = false;
const pent = [];
// `media` — MCP content blocks beside the words: the letter carries the
// picture itself, not a note about one (boot/resident-skill: params.content
// speaks the tools/call dialect and the machine lands the bytes).
const say = (text, media = []) => {
    const params = media.length === 0 ? { data: text } : { content: [{ type: 'text', text }, ...media] };
    if (!dressed) {
        pent.push(params);
        return;
    }
    send({ jsonrpc: '2.0', method: 'notifications/message', params });
};
const dress = () => {
    if (dressed) return;
    dressed = true;
    for (const params of pent.splice(0)) send({ jsonrpc: '2.0', method: 'notifications/message', params });
};

let lastChat = null;
try {
    const held = Number(fs.readFileSync(CHAT_FILE, 'utf8').trim());
    if (Number.isInteger(held)) lastChat = held;
} catch {
    // no chat learned yet — the first incoming message teaches it
}

// ---- the Bot API -------------------------------------------------
const at = method => `${API}/bot${TOKEN}/${method}`;

// Does this machine's egress ride a wall? The jail hands every call
// and resident the proxy's names, so their presence IS the answer —
// and a request that dies on the wire behind one has a cure the bare
// error never names.
const WALLED = (process.env.HTTPS_PROXY || process.env.https_proxy || '') !== '';
// A request that never LANDED, said in terms the reader can act on:
// "fetch failed" names nothing, and the commonest cause of it here is
// a wall nobody told the bridge about.
const unreached = err =>
    `${String(err?.cause?.message ?? err?.message ?? err)}${WALLED ? " — this machine's egress rides a proxy, so the wall must allow api.telegram.org (its allowlist is the operator's: infra/egress/allow.txt)" : ''}`;

// One shape for every JSON call: the HTTP verdict and Telegram's own
// `description` folded into one answer, so no caller re-derives what a
// refusal was.
async function ask(method, body) {
    try {
        const res = await fetch(at(method), {
            method: 'POST',
            headers: { 'content-type': 'application/json' },
            body: JSON.stringify(body),
        });
        const answer = await res.json().catch(() => ({}));
        return { ok: res.ok && answer.ok !== false, status: res.status, why: typeof answer.description === 'string' ? answer.description : '', result: answer.result };
    } catch (err) {
        return { ok: false, status: 0, why: unreached(err), result: null };
    }
}

const bytes = n => (n >= 1024 * 1024 ? `${(n / (1024 * 1024)).toFixed(1)} MB` : n >= 1024 ? `${Math.round(n / 1024)} KB` : `${n} B`);

// Extension → media type: the ONE table both directions read. It types
// an incoming file Telegram named by shape alone (a voice note, a
// sticker) and it decides which endpoint an outgoing one rides.
const TYPES = {
    jpg: 'image/jpeg', jpeg: 'image/jpeg', png: 'image/png', gif: 'image/gif', webp: 'image/webp', heic: 'image/heic', bmp: 'image/bmp', svg: 'image/svg+xml',
    mp4: 'video/mp4', mov: 'video/quicktime', webm: 'video/webm', mkv: 'video/x-matroska', avi: 'video/x-msvideo',
    mp3: 'audio/mpeg', m4a: 'audio/mp4', ogg: 'audio/ogg', oga: 'audio/ogg', opus: 'audio/ogg', wav: 'audio/wav', flac: 'audio/flac',
    pdf: 'application/pdf', zip: 'application/zip', gz: 'application/gzip', json: 'application/json', csv: 'text/csv', txt: 'text/plain', md: 'text/markdown', html: 'text/html',
};
const typeOf = named => TYPES[path.extname(String(named ?? '')).slice(1).toLowerCase()] ?? 'application/octet-stream';

// ---- the typing indicator ----------------------------------------
// The wait, made visible. One entry per chat: the action it wears, the
// ceiling it stops at, the timer that refreshes it.
const acting = new Map();

function beat(chat) {
    const held = acting.get(chat);
    if (held === undefined) return;
    if (Date.now() >= held.until) {
        rest(chat);
        return;
    }
    void ask('sendChatAction', { chat_id: chat, action: held.action });
    held.timer = setTimeout(() => beat(chat), TYPING_EVERY);
}

function act(chat, action = 'typing') {
    if (!Number.isInteger(chat)) return;
    const held = acting.get(chat) ?? {};
    held.action = action;
    held.until = Date.now() + TYPING_MAX;
    if (held.timer !== undefined) clearTimeout(held.timer);
    acting.set(chat, held);
    beat(chat);
}

function rest(chat) {
    const held = acting.get(chat);
    if (held === undefined) return;
    clearTimeout(held.timer);
    acting.delete(chat);
}

// ---- the send tool -----------------------------------------------
// The METHOD is the bytes' own business. `as` overrides for the two
// asks a human actually makes: an image kept lossless with its
// filename, an ogg spoken as a voice note.
const METHODS = {
    photo: { method: 'sendPhoto', field: 'photo', action: 'upload_photo' },
    video: { method: 'sendVideo', field: 'video', action: 'upload_video' },
    animation: { method: 'sendAnimation', field: 'animation', action: 'upload_video' },
    audio: { method: 'sendAudio', field: 'audio', action: 'upload_document' },
    voice: { method: 'sendVoice', field: 'voice', action: 'upload_voice' },
    document: { method: 'sendDocument', field: 'document', action: 'upload_document' },
};

function kindOf(mime, size) {
    if (mime === 'image/gif') return 'animation';
    if (mime.startsWith('image/') && mime !== 'image/svg+xml' && size <= PHOTO_MAX) return 'photo';
    if (mime.startsWith('video/')) return 'video';
    if (mime.startsWith('audio/')) return 'audio';
    return 'document';
}

// The file hands' own resolution (file.md): ~ is the home, a relative
// path anchors on the GROUND, an absolute one stands as written.
function resolveFile(named) {
    const s = String(named).trim();
    if (s === '~') return HOME;
    if (s.startsWith('~/')) return path.join(HOME, s.slice(2));
    return path.resolve(HOME, s);
}

// Telegram's 4096-char wall is the WIRE's, not the caller's: a long
// answer rides as several messages instead of coming back refused. The
// cut prefers a line boundary in the tail half — a sentence split
// mid-word reads as damage, a paragraph break reads as a page.
function pieces(text, max) {
    const parts = [];
    let left = text;
    while (left.length > max) {
        const boundary = left.slice(0, max).lastIndexOf('\n');
        const take = boundary > max / 2 ? boundary + 1 : max;
        parts.push(left.slice(0, take));
        left = left.slice(take);
    }
    if (left !== '') parts.push(left);
    return parts.length === 0 ? [text] : parts;
}

async function sendText(chat, text) {
    const parts = pieces(text, TEXT_MAX);
    for (const part of parts) {
        const answer = await ask('sendMessage', { chat_id: chat, text: part });
        if (!answer.ok) return refused(`Telegram refused the send (${answer.status}${answer.why ? `: ${answer.why}` : ''})`);
    }
    rest(chat);
    return told(`sent to chat ${chat}${parts.length > 1 ? ` in ${parts.length} messages` : ''}`);
}

async function upload(spec, chat, body, name, mime, caption) {
    const form = new FormData();
    form.append('chat_id', String(chat));
    if (caption !== '') form.append('caption', caption);
    form.append(spec.field, new Blob([body], { type: mime }), name);
    try {
        const res = await fetch(at(spec.method), { method: 'POST', body: form });
        const answer = await res.json().catch(() => ({}));
        return { ok: res.ok && answer.ok !== false, status: res.status, why: typeof answer.description === 'string' ? answer.description : '' };
    } catch (err) {
        return { ok: false, status: 0, why: unreached(err) };
    }
}

async function sendFile(chat, named, caption, forced) {
    const target = resolveFile(named);
    let stat;
    try {
        stat = fs.statSync(target);
    } catch {
        return refused(`nothing at ${target} — file takes a path on the ground (absolute, ~/… , or relative to ${HOME}); write the file first, then send it`);
    }
    if (stat.isDirectory()) return refused(`${target} is a directory — Telegram takes one file; pack a tree first (exec: zip -r out.zip <dir>) and send the archive`);
    if (stat.size > UPLOAD_MAX) return refused(`${target} is ${bytes(stat.size)} — the Bot API uploads 50 MB at most; send a link, a smaller rendering, or split it`);
    const mime = typeOf(target);
    const kind = forced ?? kindOf(mime, stat.size);
    const spec = METHODS[kind];
    if (spec === undefined) return refused(`no such kind: ${kind} — as takes one of ${Object.keys(METHODS).join(', ')}`);
    // the upload is the work the user is waiting on: the chat says so
    // in its own vocabulary while the bytes go up
    act(chat, spec.action);
    let body;
    try {
        body = fs.readFileSync(target);
    } catch (err) {
        rest(chat);
        return refused(`${target} could not be read — ${String(err?.message ?? err)}`);
    }
    const name = path.basename(target);
    const short = caption.length <= CAPTION_MAX ? caption : '';
    let lane = kind;
    let answer = await upload(spec, chat, body, name, mime, short);
    // A picture Telegram refuses on its own grounds (dimensions, ratio,
    // a codec it will not transcode) still has a road: the DOCUMENT lane
    // takes any bytes at all. Only when the kind was OURS to guess — an
    // explicit `as` is an instruction, not a preference — and the answer
    // says which lane the bytes actually took, with the refusal that
    // moved them.
    let fell = '';
    if (!answer.ok && forced === null && kind !== 'document') {
        fell = answer.why || `${answer.status}`;
        lane = 'document';
        answer = await upload(METHODS.document, chat, body, name, mime, short);
    }
    if (!answer.ok) {
        rest(chat);
        return refused(`Telegram refused the file (${answer.status}${answer.why ? `: ${answer.why}` : ''})`);
    }
    // words too long to be a caption follow the file as their own message
    if (short === '' && caption !== '') {
        const said = await sendText(chat, caption);
        if (said.isError) return said;
    }
    rest(chat);
    return told(`sent ${name} (${bytes(body.length)}, ${mime}) as a ${lane} to chat ${chat}${fell === '' ? '' : ` — Telegram would not take it as a ${kind} (${fell})`}`);
}

const TOOLS = [
    {
        name: 'send',
        description:
            'Sends a message to the Telegram chat this bridge serves — by default the chat the user last wrote from (learned from their first message; a refusal says so while nobody has written yet). Use for EVERY answer meant for the user on Telegram: they read their phone, not this window. A FILE rides the same call — file: "<path on the ground>" uploads it as the kind its bytes are (a picture arrives as a picture, a clip plays, everything else lands as a document) and text rides along as its caption. Long text is cut into several messages by itself, so never trim an answer to fit. The chat shows "typing…" from the moment the user writes until this call answers — short work needs no "on it" note.',
        inputSchema: {
            type: 'object',
            properties: {
                text: { type: 'string', description: 'the message text, plain words (Telegram renders it as-is) — the caption when a file rides along' },
                file: {
                    type: 'string',
                    description:
                        'a file to send: an absolute path, ~/… , or a path relative to the ground (the same home your exec calls and file hands work in). A file the user sent you is already there; machine media named by address ([image sha256:… ]) needs one mount first (exec(mount: ["sha256:…"], command: "cp \\"$MNT\\"/<hex> ~/out.png")).',
                },
                as: {
                    type: 'string',
                    enum: ['photo', 'video', 'animation', 'audio', 'voice', 'document'],
                    description: 'override the kind the bytes would pick: "document" keeps a picture lossless and named, "voice" makes an ogg a voice note',
                },
                chat: { type: 'integer', description: 'a specific chat id — omit for the chat the user last wrote from' },
            },
        },
    },
];

async function call(params) {
    if (params?.name !== 'send') {
        return refused(`no such tool: ${params?.name} — this bridge stands send alone`);
    }
    const args = params?.arguments ?? {};
    const text = typeof args.text === 'string' ? args.text : '';
    const file = typeof args.file === 'string' && args.file.trim() !== '' ? args.file.trim() : null;
    if (text === '' && file === null) {
        return refused('send needs text, a file, or both — the words for the user, a path for the bytes (text rides a file as its caption)');
    }
    const chat = Number.isInteger(args.chat) ? args.chat : lastChat;
    if (chat === null) {
        return refused('no chat learned yet — the user has not written to the bot; ask them to open it in Telegram and say hi, their first message teaches the chat');
    }
    if (file !== null) return sendFile(chat, file, text, typeof args.as === 'string' ? args.as : null);
    return sendText(chat, text);
}

// ---- what a message carries --------------------------------------
// The Telegram shapes, in the order a message may wear them (a photo
// message carries no document, a document message no photo) — the
// first present wins. A shape Telegram types by kind alone gets its
// media type from the downloaded name's extension.
const CARRIED = ['document', 'video', 'audio', 'voice', 'video_note', 'animation', 'sticker'];

function took(file, kind) {
    return {
        kind,
        id: file.file_id,
        unique: typeof file.file_unique_id === 'string' ? file.file_unique_id : file.file_id,
        size: Number.isInteger(file.file_size) ? file.file_size : null,
        mime: typeof file.mime_type === 'string' ? file.mime_type : null,
        name: typeof file.file_name === 'string' ? file.file_name : null,
    };
}

function carried(message) {
    // photo comes as sizes, smallest first — the LAST is the largest
    // Telegram kept, and the only one worth the ground
    if (Array.isArray(message.photo) && message.photo.length > 0) return took(message.photo[message.photo.length - 1], 'photo');
    for (const kind of CARRIED) {
        const held = message[kind];
        if (held !== null && typeof held === 'object' && typeof held.file_id === 'string') return took(held, kind);
    }
    return null;
}

// What a user can send that carries neither a file nor words — one
// line each, because a message that reaches the machine and makes no
// letter is a message the sender believes was read and nobody saw.
// The chat's own bookkeeping (a join, a pin, a title change) stays
// silent: that is the room talking, not the user.
const SAID = {
    venue: m => `a place — ${[m.venue.title, m.venue.address].filter(Boolean).join(', ')}`,
    location: m => `a location — ${m.location.latitude}, ${m.location.longitude}`,
    contact: m => `a contact — ${[m.contact.first_name, m.contact.last_name, m.contact.phone_number].filter(Boolean).join(' ')}`,
    poll: m => `a poll — ${m.poll.question ?? ''}`,
    dice: m => `a dice roll — ${m.dice.value}`,
};

function spoken(message) {
    for (const [kind, tell] of Object.entries(SAID)) {
        if (message[kind] === null || typeof message[kind] !== 'object') continue;
        try {
            return tell(message);
        } catch {
            return `a ${kind}`;
        }
    }
    return null;
}

// A name off the world lands as ONE leaf under our own directory: a
// separator or a leading .. in it would write outside the ground.
function leaf(named) {
    const bare = path.basename(String(named ?? '').replace(/[\\/]+/g, '-')).replace(/^\.+/, '').trim();
    return bare === '' ? 'file' : bare.slice(0, 120);
}

// The same wall for an id used as a DIRECTORY name — nothing but the
// safe alphabet, and never a leading dot (".." is two legal characters
// that climb out of the ground).
const slug = named => String(named ?? '').replace(/[^A-Za-z0-9._-]/g, '_').replace(/^\.+/, '').slice(0, 80) || 'file';

// The file's own directory, named by Telegram's stable per-file id: two
// photos both called "photo.jpg" keep both sets of bytes, the sender's
// filename survives whole (an archive to unpack, a pdf to read), and a
// re-sent file lands on itself instead of piling up copies.
async function land(found) {
    const meta = await ask('getFile', { file_id: found.id });
    const where = meta.result?.file_path;
    if (!meta.ok || typeof where !== 'string') return { why: meta.why || `getFile refused (${meta.status})` };
    let body;
    try {
        const res = await fetch(`${API}/file/bot${TOKEN}/${where}`);
        if (!res.ok) return { why: `the download refused (${res.status})` };
        body = Buffer.from(await res.arrayBuffer());
    } catch (err) {
        return { why: String(err?.message ?? err) };
    }
    const name = leaf(found.name ?? `${found.kind}${path.extname(where)}`);
    const dir = path.join(IN_DIR, slug(found.unique));
    try {
        fs.mkdirSync(dir, { recursive: true });
        const target = path.join(dir, name);
        fs.writeFileSync(target, body);
        return { path: target, body, mime: found.mime ?? typeOf(name) };
    } catch (err) {
        return { why: `the ground refused the file — ${String(err?.message ?? err)}` };
    }
}

// What the companion may PERCEIVE inline. Sound rides too: the wire
// drops what a provider cannot hear, but the bytes land on the blob
// plane all the same — the window plays the voice note the user sent.
function shown(landed) {
    if (landed.body.length > INLINE_MAX) return [];
    if (SEEABLE.has(landed.mime)) return [{ type: 'image', data: landed.body.toString('base64'), mimeType: landed.mime }];
    if (landed.mime.startsWith('audio/')) return [{ type: 'audio', data: landed.body.toString('base64'), mimeType: landed.mime }];
    return [];
}

function learn(chat) {
    if (!Number.isInteger(chat) || chat === lastChat) return;
    lastChat = chat;
    try {
        fs.writeFileSync(CHAT_FILE, String(chat));
    } catch {
        // the ground refused — the chat still serves this life
    }
}

async function relay(message) {
    const found = carried(message);
    const words = typeof message.text === 'string' ? message.text : typeof message.caption === 'string' ? message.caption : '';
    const other = found === null && words === '' ? spoken(message) : null;
    // the room's own bookkeeping — nothing was said and nothing sent
    if (found === null && words === '' && other === null) return;
    const chat = message.chat?.id;
    learn(chat);
    // the machine HAS it: the chat says so from this instant — before
    // the download, before the wake, before the first token
    act(chat);
    const from = message.from?.username || message.from?.first_name || 'someone';
    if (other !== null) {
        say(`${from} sends ${other}`);
        return;
    }
    // the sender on its own line, the words verbatim below —
    // the reader (and any trigger) sees the message itself
    if (found === null) {
        say(`${from} writes:\n${words}`);
        return;
    }
    const landed = await land(found);
    if (landed.why !== undefined) {
        // A file too big for the Bot API (its own 20 MB wall on
        // downloads) lands here: the companion learns WHAT was sent and
        // WHY it is not on the disk, instead of the file vanishing
        // between the wires.
        const called = `${found.name ? ` "${found.name}"` : ''}${found.size === null ? '' : ` (${bytes(found.size)})`}`;
        say(`${from} sends a ${found.kind}${called} — it did not land: ${landed.why}${words === '' ? '' : `\n${words}`}`);
        return;
    }
    const note = `${from} sends a ${found.kind} — ${landed.path} (${landed.mime}, ${bytes(landed.body.length)})`;
    say(words === '' ? note : `${note}\n${words}`, shown(landed));
}

// ---- the poll ----------------------------------------------------
// getUpdates with a long-poll timeout: the process sleeps on the wire,
// never on a clock (§17 — the machine's wearers park for free; this
// process is the one paying attention).
//
// AND THE POLL SPEAKS WHEN IT CANNOT POLL. A bridge that cannot reach
// Telegram looks EXACTLY like a bridge nobody wrote to — the chat is
// quiet either way — and the difference is an hour of blind digging
// (observed live behind an allowlist proxy: every request died at
// connect, the loop breathed and re-asked forever, and the machine's
// only evidence was silence). So each spell of trouble is SAID ONCE,
// with its own cause, and the recovery is said too; a flapping wire
// costs two letters, not a flood. The words go to the wearers AND to
// stderr, which is this life's chatter log — the one place a reader
// looks when the mailbox is empty.
let offset = 0;
let ailing = '';
const note = text => process.stderr.write(`${text}\n`);
// `kind` is the fault's IDENTITY, not its wording: the same wall
// failing a thousand times says nothing new. Answers whether this
// spell was FRESH — the caller breathes shortly on a first fault (a
// blip clears in seconds) and slowly while it persists.
const ails = (kind, text) => {
    if (ailing === kind) return false;
    ailing = kind;
    note(text);
    say(text);
    return true;
};
const heals = () => {
    if (ailing === '') return;
    ailing = '';
    const back = 'Telegram answers again — polling resumed.';
    note(back);
    say(back);
};
// A refusal from the WALL is not a refusal from Telegram: an egress
// proxy answers 403 with a page, Telegram with its own JSON. Naming
// the wrong one sends the reader to reseal a token that was never
// wrong.
const walled = text => `the machine's egress proxy refused it — the wall must allow api.telegram.org (its allowlist is the operator's: infra/egress/allow.txt)${text ? `; it said: ${text.slice(0, 200)}` : ''}`;
async function poll() {
    for (;;) {
        try {
            const res = await fetch(`${API}/bot${TOKEN}/getUpdates?timeout=25&offset=${offset}`);
            if (res.status === 401 || res.status === 403) {
                const said = await res.text().catch(() => '');
                let body;
                try {
                    body = JSON.parse(said);
                } catch {
                    body = null;
                }
                // Telegram's own refusal carries its JSON; anything else
                // on this status is something standing between us
                const fresh =
                    body?.ok === false
                        ? ails('token', `Telegram refused the token (${res.status}${body.description ? `: ${body.description}` : ''}) — reseal TG_TOKEN in the vault, then kill("telegram/bridge"); the wear stands it fresh. Polling continues.`)
                        : ails('wall', `Telegram is unreachable (${res.status}): ${walled(said)}. Polling continues.`);
                await new Promise(r => setTimeout(r, fresh ? 3_000 : 30_000));
                continue;
            }
            if (!res.ok) {
                const fresh = ails(`status:${res.status}`, `Telegram answered ${res.status} to the poll — nothing the user sends arrives until it clears; polling continues.`);
                await new Promise(r => setTimeout(r, fresh ? 3_000 : 30_000));
                continue;
            }
            heals();
            const body = await res.json().catch(() => ({}));
            for (const update of Array.isArray(body.result) ? body.result : []) {
                if (Number.isInteger(update.update_id)) offset = Math.max(offset, update.update_id + 1);
                const message = update.message ?? update.edited_message;
                if (message === null || typeof message !== 'object') continue;
                // one message at a time, in the order they were sent: a
                // download must not let the next letter overtake it
                await relay(message);
            }
        } catch (err) {
            // The wire never reached Telegram at all — a DNS miss, a
            // clamped net, a proxy that is not there. This is the mute
            // failure the machine must never keep to itself.
            const fresh = ails('wire', `the bridge cannot reach ${API}: ${unreached(err)}. Nothing the user sends arrives while this stands; polling keeps retrying.`);
            await new Promise(r => setTimeout(r, fresh ? 3_000 : 30_000));
        }
    }
}

if (TOKEN === '') {
    // stood outside an install (no keyring name) — say it and serve
    // the refusal through every tool call rather than dying silent
    say('TG_TOKEN is not on this env — seal it to the vault under that exact name and re-stand the bridge.');
} else {
    void poll();
}

// ---- the wire ----------------------------------------------------
process.stdin.setEncoding('utf8');
let buf = '';
process.stdin.on('data', chunk => {
    buf += chunk;
    for (;;) {
        const cut = buf.indexOf('\n');
        if (cut < 0) break;
        const line = buf.slice(0, cut);
        buf = buf.slice(cut + 1);
        if (line.trim() === '') continue;
        let msg;
        try {
            msg = JSON.parse(line);
        } catch {
            continue;
        }
        if (msg.method === 'initialize')
            ok(msg.id, { protocolVersion: msg.params?.protocolVersion || '2025-06-18', capabilities: { tools: {} }, serverInfo: { name: 'telegram-bridge', version: '1' } });
        else if (msg.method === 'tools/list') {
            ok(msg.id, { tools: TOOLS });
            // the roster is served — queued voice flushes on the next
            // tick, after the machine has read the list
            setTimeout(dress, 50);
        }
        else if (msg.method === 'tools/call') void call(msg.params).then(result => ok(msg.id, result));
        else if (msg.method === 'ping') ok(msg.id, {});
        else if (msg.id !== undefined && msg.id !== null) send({ jsonrpc: '2.0', id: msg.id, error: { code: -32601, message: `unknown method: ${msg.method}` } });
    }
});
