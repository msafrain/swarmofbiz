'use strict';
// Blogger serves public feeds as JSONP. This avoids exposing API credentials.
const BLOG = 'https://swarmofbiz.blogspot.com';
const CATEGORIES = ['Automotive','Beauty & Wellness','Business Services','Design','Education','Events','Finance','Home','Logistics','Marketing','Photography','Property','Technology','Transportation','Videography'];
const BATCH = 150;
const cardsEl = document.getElementById('cards');
const statusEl = document.getElementById('status');
const titleEl = document.getElementById('page-title');
const navEl = document.getElementById('categories');

function categoryFromUrl(){const p=new URLSearchParams(location.search);return CATEGORIES.find(x=>x===p.get('category'))||null}
function showStatus(message){statusEl.textContent=message}
function jsonp(url){return new Promise((resolve,reject)=>{
  const callback='sobFeed_'+Date.now()+'_'+Math.random().toString(36).slice(2);
  const script=document.createElement('script');
  const timer=setTimeout(()=>finish(new Error('Blogger did not respond.')),15000);
  function finish(error,value){clearTimeout(timer);script.remove();delete window[callback];error?reject(error):resolve(value)}
  window[callback]=data=>finish(null,data);
  script.onerror=()=>finish(new Error('Could not load the Blogger feed.'));
  script.src=url+(url.includes('?')?'&':'?')+'alt=json-in-script&callback='+callback;
  document.head.appendChild(script);
})}
function feedPath(category){return BLOG+'/feeds/posts/default'+(category?'/-/'+encodeURIComponent(category):'')}
async function loadEntries(category){
  const all=[];let start=1;
  while(true){
    const data=await jsonp(feedPath(category)+'?start-index='+start+'&max-results='+(category?BATCH:6));
    const entries=data.feed?.entry||[];
    all.push(...entries);
    if(!category||entries.length<BATCH)break;
    start+=entries.length;
    if(start>20000)throw new Error('Archive is too large to load in one request.');
  }
  return all;
}
function postLink(entry){return entry.link?.find(l=>l.rel==='alternate'&&l.type==='text/html')?.href||BLOG}
function imageFromPost(entry){
  // Prefer the first full-size image in the post; Blogger thumbnails can be small.
  const html=entry.content?.$t||entry.summary?.$t||'';
  const doc=new DOMParser().parseFromString(html,'text/html');
  const img=doc.querySelector('img');
  if(img){const href=img.closest('a')?.href;return href&&/^https?:\/\//.test(href)?href:img.getAttribute('src')}
  return entry.media$thumbnail?.url?.replace(/\/s\d+(?:-c)?\//,'/s1200/')||null;
}
function render(entries){
  cardsEl.replaceChildren();let count=0;
  for(const entry of entries){
    const src=imageFromPost(entry);if(!src)continue;
    const a=document.createElement('a');a.className='card';a.href=postLink(entry);a.target='_blank';a.rel='noopener noreferrer';
    a.setAttribute('aria-label',entry.title?.$t||'View business card');
    const img=document.createElement('img');img.src=src;img.alt=entry.title?.$t||'Business card';img.loading=count<2?'eager':'lazy';
    a.append(img);cardsEl.append(a);count++;
  }
  showStatus(count?'': 'No business cards published yet.');
}
function renderNav(selected){for(const cat of CATEGORIES){const a=document.createElement('a');a.textContent=cat;a.href='?category='+encodeURIComponent(cat);if(cat===selected){a.className='active';a.setAttribute('aria-current','page')}navEl.append(a)}}
async function init(){const category=categoryFromUrl();renderNav(category);titleEl.textContent=category?category.toUpperCase():'RECENT CARDS';document.title=(category?category+' — ':'')+'Swarm of Biz';try{const entries=await loadEntries(category);render(entries)}catch(e){console.error(e);showStatus('Unable to load cards. Check that the Blogger site and its feed are public.')}}
init();
