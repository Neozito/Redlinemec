/* assets/js/app.js
   Lógica simples usando localStorage para demo:
   - users: [{email, password, name}]
   - clients: [{id,name,phone,email,note,created_at}]
   - appointments: [{id,client_id,service, start, end, price, status, notes}]
*/

/* util */
const uid = ()=>Date.now().toString(36)+Math.random().toString(36).slice(2,8);

function storageGet(key){
  try{return JSON.parse(localStorage.getItem(key) || 'null')}
  catch(e){return null}
}
function storageSet(key,val){ localStorage.setItem(key, JSON.stringify(val)) }

/* bootstrap seed data (executa só se vazio) */
function ensureSeed(){
  if(!storageGet('users')){
    storageSet('users', [{email:'admin@salon.local', password:'admin123', name:'Admin'}]);
  }
  if(!storageGet('clients')){
    storageSet('clients', [
      {id:'c1', name:'Raphael Bruno Correia', phone:'45984069119', email:'neoartepp@gmail.com', note:'Cliente VIP - SEO agencia neo arte', created_at: new Date().toISOString()}
    ]);
  }
  if(!storageGet('appointments')){
    const start = new Date('2025-11-14T10:30:00').toISOString();
    const end = new Date('2025-11-14T11:15:00').toISOString();
    storageSet('appointments', [
      {id:'a1', client_id:'c1', service:'Corte + Barba', start, end, price:50.00, status:'scheduled', notes:'Observações do cliente', created_at: new Date().toISOString()}
    ]);
  }
}
ensureSeed();

/* auth helpers */
function login(email,password){
  const users = storageGet('users') || [];
  return users.find(u => u.email === email && u.password === password) || null;
}
function logout(){
  localStorage.removeItem('logged_user');
  window.location.href = 'index.html';
}
function setLogged(user){
  localStorage.setItem('logged_user', JSON.stringify(user));
}
function getLogged(){
  return storageGet('logged_user');
}

/* clients API */
const ClientsAPI = {
  list(q){
    const all = storageGet('clients') || [];
    if(!q) return all;
    q = q.toLowerCase();
    return all.filter(c => (c.name||'').toLowerCase().includes(q) || (c.phone||'').includes(q) || (c.email||'').toLowerCase().includes(q));
  },
  get(id){ return (storageGet('clients')||[]).find(c=>c.id===id) || null },
  save(data){
    let clients = storageGet('clients')||[];
    if(data.id){
      clients = clients.map(c => c.id===data.id ? {...c,...data} : c);
    } else {
      data.id = uid(); data.created_at = new Date().toISOString();
      clients.push(data);
    }
    storageSet('clients', clients);
    return data;
  },
  remove(id){
    let clients = storageGet('clients')||[];
    clients = clients.filter(c => c.id !== id);
    storageSet('clients', clients);
  }
};

/* appointments API */
const ApptsAPI = {
  list(){ return storageGet('appointments') || [] },
  listEventsForCalendar(){
    return (storageGet('appointments')||[]).map(a => ({
      id:a.id,
      title: (ClientsAPI.get(a.client_id)?.name || 'Cliente') + ' — ' + a.service,
      start: a.start,
      end: a.end,
      extendedProps: a
    }));
  },
  save(data){
    let appts = storageGet('appointments')||[];
    if(data.id){
      appts = appts.map(a=> a.id===data.id ? {...a,...data} : a);
    } else {
      data.id = uid();
      data.created_at = new Date().toISOString();
      appts.push(data);
    }
    storageSet('appointments', appts);
    return data;
  },
  remove(id){
    let appts = storageGet('appointments')||[];
    appts = appts.filter(a=>a.id !== id);
    storageSet('appointments', appts);
  }
};

/* expose to window for pages */
window._Salon = {
  login, logout, setLogged, getLogged, ClientsAPI, ApptsAPI, uid
};
