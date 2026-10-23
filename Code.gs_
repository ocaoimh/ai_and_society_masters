/**
 * BSF live polling backend for the reveal.js deck.
 *
 * Setup (once):
 * 1. Create a new Google Sheet. Extensions > Apps Script.
 * 2. Paste this file into Code.gs and add an HTML file named "Participant"
 *    with the contents of Participant.html.
 * 3. Change PRESENTER_KEY below to a word only you know.
 * 4. Deploy > New deployment > Web app.
 *      Execute as: Me
 *      Who has access: Anyone
 *    Copy the /exec URL into CONFIG.apiUrl in the deck.
 *
 * Participants open the /exec URL (or scan the QR code): they see whichever
 * question the deck is currently showing. The deck sets the active question
 * and reads results through JSONP calls to the same URL.
 */

var PRESENTER_KEY = 'change-me';
var SHEET_NAME = 'responses';
var MAX_TEXT = 200;

function doGet(e) {
  var p = (e && e.parameter) || {};
  if (p.api) return api_(p);
  return HtmlService.createHtmlOutputFromFile('Participant')
    .setTitle('Your turn')
    .addMetaTag('viewport', 'width=device-width, initial-scale=1');
}

/* ---------- presenter API (JSONP) ---------- */

function api_(p) {
  var out;
  try {
    if (p.api === 'results') {
      out = results_(p.q);
    } else if (p.api === 'setActive') {
      checkKey_(p.key);
      var q = JSON.parse(p.question);
      setActive_(q);
      out = { ok: true, active: q.id };
    } else if (p.api === 'clearActive') {
      checkKey_(p.key);
      setActive_(null);
      out = { ok: true };
    } else if (p.api === 'reset') {
      checkKey_(p.key);
      sheet_().clearContents();
      setActive_(null);
      out = { ok: true };
    } else {
      out = { error: 'unknown api' };
    }
  } catch (err) {
    out = { error: String(err.message || err) };
  }
  var json = JSON.stringify(out);
  if (p.callback && /^[A-Za-z0-9_$.]+$/.test(p.callback)) {
    return ContentService.createTextOutput(p.callback + '(' + json + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
  return ContentService.createTextOutput(json)
    .setMimeType(ContentService.MimeType.JSON);
}

function checkKey_(key) {
  if (key !== PRESENTER_KEY) throw new Error('bad presenter key');
}

function setActive_(q) {
  var value = q ? JSON.stringify({ id: q.id, type: q.type, prompt: q.prompt, options: q.options || [] }) : '';
  PropertiesService.getScriptProperties().setProperty('ACTIVE', value);
  CacheService.getScriptCache().put('ACTIVE', value || 'none', 21600);
}

function results_(qid) {
  if (!qid) throw new Error('missing q');
  var rows = sheet_().getDataRange().getValues();
  var latest = {}; // one answer per participant per question, last one wins
  rows.forEach(function (r) {
    if (r[1] === qid) latest[r[2]] = { t: r[0], a: String(r[3]) };
  });
  var counts = {};
  var texts = [];
  Object.keys(latest).forEach(function (id) {
    var a = latest[id].a;
    counts[a] = (counts[a] || 0) + 1;
    texts.push({ t: new Date(latest[id].t).getTime(), a: a });
  });
  texts.sort(function (x, y) { return y.t - x.t; });
  return { q: qid, n: Object.keys(latest).length, counts: counts, texts: texts.slice(0, 80).map(function (x) { return x.a; }) };
}

/* ---------- participant API (google.script.run) ---------- */

function getActive() {
  var v = CacheService.getScriptCache().get('ACTIVE');
  if (v === null) {
    v = PropertiesService.getScriptProperties().getProperty('ACTIVE') || '';
    CacheService.getScriptCache().put('ACTIVE', v || 'none', 21600);
  }
  return v && v !== 'none' ? JSON.parse(v) : null;
}

function submit(qid, answer, clientId) {
  var active = getActive();
  if (!active || active.id !== qid) throw new Error('This question is closed.');
  answer = String(answer || '').trim().slice(0, MAX_TEXT);
  if (!answer) throw new Error('Empty answer.');
  if (active.type === 'mcq') {
    var i = Number(answer);
    if (!(i >= 0 && i < active.options.length)) throw new Error('Invalid option.');
    answer = String(i);
  }
  clientId = String(clientId || '').slice(0, 64);
  var lock = LockService.getScriptLock();
  lock.waitLock(10000);
  try {
    // Leading apostrophe keeps Sheets from turning text into dates or formulas.
    sheet_().appendRow([new Date(), qid, "'" + clientId, "'" + answer]);
  } finally {
    lock.releaseLock();
  }
  return true;
}

function sheet_() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  return ss.getSheetByName(SHEET_NAME) || ss.insertSheet(SHEET_NAME);
}
