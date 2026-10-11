import React, { useState, useEffect, useMemo } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, collection, addDoc, onSnapshot, updateDoc, doc, serverTimestamp } from 'firebase/firestore';
import { Search, Globe, Menu, X, ChevronRight, Package, ShieldCheck, TrendingUp, Mail, MapPin, Phone, MessageSquare, LogIn, CheckCircle2, Clock, Check } from 'lucide-react';

const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const app = Object.keys(firebaseConfig).length > 0 ? initializeApp(firebaseConfig) : { name: 'mock-app' };
const auth = Object.keys(firebaseConfig).length > 0 ? getAuth(app) : null;
const db = Object.keys(firebaseConfig).length > 0 ? getFirestore(app) : null;
const appId = typeof __app_id !== 'undefined' ? __app_id : 'indra-export-app';

const DICTIONARY = {
  en: {
    nav: { home: 'Home', products: 'Products', about: 'About INDRA', contact: 'Contact Us', admin: 'Admin Login' },
    hero: { title: 'Connecting Indian Quality with Russian Markets', subtitle: 'Reliable sourcing, transparent communication, and professional international trade.', cta1: 'Request a Quotation', cta2: 'Explore Our Products' },
    features: { title: 'Why Choose INDRA', f1Title: 'Verified Quality', f1Desc: 'Strict quality control & supplier coordination.', f2Title: 'Transparent Trade', f2Desc: 'Clear communication and competitive quotations.', f3Title: 'End-to-End Export', f3Desc: 'From sourcing to shipment coordination.' },
    process: { title: 'Our Export Process', s1: 'Submit Enquiry', s2: 'Receive Quotation', s3: 'Confirm Terms', s4: 'Shipment' },
    products: { title: 'Product Catalog', search: 'Search products...', filterAll: 'All Categories', reqQuote: 'Request Quote', noResults: 'No products found.' },
    about: { title: 'About INDRA', p1: 'INDRA is a premier Indian export and international trading company dedicated to fulfilling the sourcing needs of the Russian market.', p2: 'Our mission is to build long-term business relationships based on trust, quality assurance, and seamless international trade.' },
    contact: { title: 'Contact Us', address: 'Bengaluru, Karnataka, India', email: 'export@indra-trade.com', phone: '+91 98765 43210' },
    form: { title: 'Request a Quotation', name: 'Buyer/Company Name', contactPerson: 'Contact Person', email: 'Business Email', phone: 'WhatsApp Number', country: 'Country', product: 'Product of Interest', qty: 'Required Quantity', unit: 'Unit (e.g., kg, tonnes)', dest: 'Destination Port/City in Russia', notes: 'Additional Specifications', submit: 'Submit Enquiry', success: 'Enquiry submitted successfully. Reference: ', sending: 'Sending...' },
    admin: { title: 'Admin Dashboard', login: 'Login to Dashboard', passPlaceholder: 'Enter Admin Password (indra2026)', error: 'Invalid password', list: 'Enquiry List', status: 'Status', markContacted: 'Mark Contacted', markConfirmed: 'Mark Confirmed', mockEmail: 'Email Notification Sent', mockWa: 'WhatsApp Notification Sent' },
    footer: { text: '© 2026 INDRA International Trade. All rights reserved.' }
  },
  ru: {
    nav: { home: 'Главная', products: 'Продукты', about: 'О нас', contact: 'Контакты', admin: 'Вход для админа' },
    hero: { title: 'Соединяем индийское качество с российскими рынками', subtitle: 'Надежный сорсинг, прозрачная коммуникация и профессиональная международная торговля.', cta1: 'Запросить предложение', cta2: 'Изучить продукты' },
    features: { title: 'Почему выбирают INDRA', f1Title: 'Проверенное качество', f1Desc: 'Строгий контроль качества и координация.', f2Title: 'Прозрачная торговля', f2Desc: 'Четкая коммуникация и конкурентные цены.', f3Title: 'Экспорт под ключ', f3Desc: 'От поиска до координации поставок.' },
    process: { title: 'Процесс экспорта', s1: 'Отправить запрос', s2: 'Получить предложение', s3: 'Подтвердить условия', s4: 'Отгрузка' },
    products: { title: 'Каталог продукции', search: 'Поиск товаров...', filterAll: 'Все категории', reqQuote: 'Запросить', noResults: 'Товары не найдены.' },
    about: { title: 'Об INDRA', p1: 'INDRA — ведущая индийская экспортная и торговая компания, специализирующаяся на потребностях российского рынка.', p2: 'Наша миссия — построение долгосрочных отношений на основе доверия, гарантии качества и бесперебойной торговли.' },
    contact: { title: 'Свяжитесь с нами', address: 'Бангалор, Карнатака, Индия', email: 'export@indra-trade.com', phone: '+91 98765 43210' },
    form: { title: 'Запросить коммерческое предложение', name: 'Название компании/Покупатель', contactPerson: 'Контактное лицо', email: 'Рабочий Email', phone: 'Номер WhatsApp', country: 'Страна', product: 'Интересующий продукт', qty: 'Требуемое количество', unit: 'Ед. изм. (кг, тонны)', dest: 'Порт/Город назначения в РФ', notes: 'Доп. требования', submit: 'Отправить запрос', success: 'Запрос успешно отправлен. Номер: ', sending: 'Отправка...' },
    admin: { title: 'Панель администратора', login: 'Войти в панель', passPlaceholder: 'Введите пароль', error: 'Неверный пароль', list: 'Список запросов', status: 'Статус', markContacted: 'Связались', markConfirmed: 'Подтверждено', mockEmail: 'Email отправлен', mockWa: 'WhatsApp отправлен' },
    footer: { text: '© 2026 INDRA International Trade. Все права защищены.' }
  }
};

const MOCK_PRODUCTS = [
  { id: 'p1', category: 'Agriculture', img: 'https://images.unsplash.com/photo-1596040033229-a9821ebd058d?auto=format&fit=crop&q=80&w=400', en: { name: 'Premium Assam Tea', desc: 'High-quality black tea leaves sourced directly from Assam estates.' }, ru: { name: 'Премиальный Ассамский чай', desc: 'Высококачественные листья черного чая напрямую с плантаций Ассама.' } },
  { id: 'p2', category: 'Spices', img: 'https://images.unsplash.com/photo-1596040033229-a9821ebd058d?auto=format&fit=crop&q=80&w=400', en: { name: 'Malabar Black Pepper', desc: 'Authentic Indian black pepper, sorted and graded for export.' }, ru: { name: 'Малабарский черный перец', desc: 'Аутентичный индийский черный перец, отсортированный для экспорта.' } },
  { id: 'p3', category: 'Textiles', img: 'https://images.unsplash.com/photo-1596040033229-a9821ebd058d?auto=format&fit=crop&q=80&w=400', en: { name: 'Organic Cotton Yarn', desc: '100% organic cotton yarn suitable for industrial weaving.' }, ru: { name: 'Органическая хлопчатобумажная пряжа', desc: '100% органическая пряжа, подходящая для промышленного ткачества.' } },
  { id: 'p4', category: 'Chemicals', img: 'https://images.unsplash.com/photo-1596040033229-a9821ebd058d?auto=format&fit=crop&q=80&w=400', en: { name: 'Industrial Grade APIs', desc: 'Certified Active Pharmaceutical Ingredients for formulation.' }, ru: { name: 'Промышленные АФИ', desc: 'Сертифицированные активные фармацевтические ингредиенты.' } },
];

export default function App() {
  const [lang, setLang] = useState('en');
  const [view, setView] = useState('home');
  const [user, setUser] = useState(null);
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const [quoteProduct, setQuoteProduct] = useState(null);
  const [toastMessage, setToastMessage] = useState('');

  const t = DICTIONARY[lang];

  useEffect(() => {
    if (!auth) return;
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) {
        console.error("Auth error:", err);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  const showToast = (msg) => {
    setToastMessage(msg);
    setTimeout(() => setToastMessage(''), 4000);
  };

  const changeView = (v) => {
    setView(v);
    setIsMenuOpen(false);
    window.scrollTo(0, 0);
  };

  const Toast = () => toastMessage ? (
    <div className="fixed bottom-4 left-1/2 -translate-x-1/2 z-50 bg-slate-900 text-white px-6 py-3 rounded-xl shadow-2xl flex items-center gap-3 animate-fade-in">
      <CheckCircle2 className="text-orange-500 w-5 h-5" />
      <span>{toastMessage}</span>
    </div>
  ) : null;

  const Navbar = () => (
    <nav className="sticky top-0 z-40 bg-white/95 backdrop-blur-md border-b border-slate-100 shadow-sm">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between h-20">
          <div className="flex items-center cursor-pointer" onClick={() => changeView('home')}>
            <div className="w-10 h-10 bg-slate-900 rounded-lg flex items-center justify-center mr-3">
              <Globe className="text-orange-500 w-6 h-6" />
            </div>
            <span className="text-2xl font-bold tracking-wider text-slate-900">INDRA</span>
          </div>
          
          <div className="hidden md:flex items-center space-x-8">
            {['home', 'products', 'about', 'contact'].map(v => (
              <button key={v} onClick={() => changeView(v)} className={`text-sm font-medium transition-colors ${view === v ? 'text-orange-600' : 'text-slate-600 hover:text-slate-900'}`}>
                {t.nav[v]}
              </button>
            ))}
            <div className="flex items-center bg-slate-100 p-1 rounded-lg">
              <button onClick={() => setLang('en')} className={`px-3 py-1 text-xs font-bold rounded-md ${lang === 'en' ? 'bg-white shadow text-slate-900' : 'text-slate-500'}`}>EN</button>
              <button onClick={() => setLang('ru')} className={`px-3 py-1 text-xs font-bold rounded-md ${lang === 'ru' ? 'bg-white shadow text-slate-900' : 'text-slate-500'}`}>RU</button>
            </div>
            <button onClick={() => changeView('admin')} className="text-slate-400 hover:text-slate-900">
               <LogIn className="w-5 h-5" />
            </button>
          </div>

          <div className="md:hidden flex items-center">
             <button onClick={() => setIsMenuOpen(!isMenuOpen)} className="text-slate-900">
               {isMenuOpen ? <X className="w-6 h-6" /> : <Menu className="w-6 h-6" />}
             </button>
          </div>
        </div>
      </div>
      
      {/* Mobile Menu */}
      {isMenuOpen && (
        <div className="md:hidden bg-white border-t border-slate-100 px-4 pt-2 pb-4 space-y-1 shadow-lg">
          {['home', 'products', 'about', 'contact', 'admin'].map(v => (
             <button key={v} onClick={() => changeView(v)} className="block w-full text-left px-3 py-3 rounded-md text-base font-medium text-slate-900 hover:bg-slate-50">
               {t.nav[v]}
             </button>
          ))}
          <div className="px-3 py-3 flex gap-2">
            <button onClick={() => { setLang('en'); setIsMenuOpen(false); }} className={`flex-1 py-2 text-sm font-bold rounded-md ${lang === 'en' ? 'bg-slate-900 text-white' : 'bg-slate-100 text-slate-600'}`}>English</button>
            <button onClick={() => { setLang('ru'); setIsMenuOpen(false); }} className={`flex-1 py-2 text-sm font-bold rounded-md ${lang === 'ru' ? 'bg-slate-900 text-white' : 'bg-slate-100 text-slate-600'}`}>Русский</button>
          </div>
        </div>
      )}
    </nav>
  );

  const HomeView = () => (
    <div className="animate-in fade-in duration-500">
      {/* Hero */}
      <div className="relative bg-slate-900 text-white overflow-hidden">
        <div className="absolute inset-0 bg-[url('https://images.unsplash.com/photo-1494412574643-ff11b0a5c1c3?auto=format&fit=crop&q=80&w=2000')] bg-cover bg-center opacity-20 mix-blend-overlay"></div>
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-24 lg:py-40 relative z-10 text-center">
          <h1 className="text-4xl md:text-6xl font-extrabold tracking-tight mb-6 max-w-4xl mx-auto">
            {t.hero.title}
          </h1>
          <p className="text-lg md:text-xl text-slate-300 mb-10 max-w-2xl mx-auto">
            {t.hero.subtitle}
          </p>
          <div className="flex flex-col sm:flex-row justify-center gap-4">
            <button onClick={() => setQuoteProduct('General')} className="bg-orange-500 hover:bg-orange-600 text-white px-8 py-4 rounded-xl font-bold transition-all shadow-lg shadow-orange-500/30 flex items-center justify-center gap-2">
              {t.hero.cta1} <ChevronRight className="w-5 h-5" />
            </button>
            <button onClick={() => changeView('products')} className="bg-white/10 hover:bg-white/20 text-white border border-white/20 px-8 py-4 rounded-xl font-bold transition-all flex items-center justify-center">
              {t.hero.cta2}
            </button>
          </div>
        </div>
      </div>

      {/* Features */}
      <div className="py-20 bg-slate-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <h2 className="text-3xl font-bold text-center text-slate-900 mb-16">{t.features.title}</h2>
          <div className="grid md:grid-cols-3 gap-10">
            {[
              { icon: ShieldCheck, title: t.features.f1Title, desc: t.features.f1Desc },
              { icon: TrendingUp, title: t.features.f2Title, desc: t.features.f2Desc },
              { icon: Package, title: t.features.f3Title, desc: t.features.f3Desc }
            ].map((f, i) => (
              <div key={i} className="bg-white p-8 rounded-2xl shadow-sm border border-slate-100 text-center hover:shadow-xl transition-shadow">
                <div className="w-16 h-16 bg-orange-100 rounded-2xl flex items-center justify-center mx-auto mb-6 text-orange-600">
                  <f.icon className="w-8 h-8" />
                </div>
                <h3 className="text-xl font-bold text-slate-900 mb-3">{f.title}</h3>
                <p className="text-slate-600 leading-relaxed">{f.desc}</p>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );

  const ProductsView = () => {
    const [searchTerm, setSearchTerm] = useState('');
    const [category, setCategory] = useState('All');
    
    const categories = ['All', ...new Set(MOCK_PRODUCTS.map(p => p.category))];
    const filtered = MOCK_PRODUCTS.filter(p => {
      const matchCat = category === 'All' || p.category === category;
      const matchSearch = p[lang].name.toLowerCase().includes(searchTerm.toLowerCase());
      return matchCat && matchSearch;
    });

    return (
      <div className="py-12 px-4 max-w-7xl mx-auto min-h-screen">
        <h1 className="text-4xl font-bold text-slate-900 mb-8">{t.products.title}</h1>
        
        <div className="flex flex-col md:flex-row gap-4 mb-10">
          <div className="relative flex-1">
            <Search className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-400 w-5 h-5" />
            <input 
              type="text" 
              placeholder={t.products.search}
              value={searchTerm}
              onChange={e => setSearchTerm(e.target.value)}
              className="w-full pl-12 pr-4 py-3 bg-white border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none shadow-sm"
            />
          </div>
          <select 
            value={category} 
            onChange={e => setCategory(e.target.value)}
            className="px-4 py-3 bg-white border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 focus:outline-none shadow-sm min-w-[200px]"
          >
            {categories.map(c => <option key={c} value={c}>{c === 'All' ? t.products.filterAll : c}</option>)}
          </select>
        </div>

        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-8">
          {filtered.length > 0 ? filtered.map(p => (
            <div key={p.id} className="bg-white rounded-2xl overflow-hidden shadow-sm border border-slate-100 hover:shadow-xl transition-all group">
              <div className="h-48 overflow-hidden relative bg-slate-100">
                 {/* Fake image with colored background placeholder since real URL might block or load slow */}
                 <div className="absolute inset-0 bg-slate-200 flex items-center justify-center text-slate-400">
                    <Package className="w-12 h-12" />
                 </div>
              </div>
              <div className="p-6">
                <span className="text-xs font-bold text-orange-500 tracking-wider uppercase mb-2 block">{p.category}</span>
                <h3 className="text-xl font-bold text-slate-900 mb-2">{p[lang].name}</h3>
                <p className="text-slate-600 text-sm mb-6 line-clamp-2">{p[lang].desc}</p>
                <button 
                  onClick={() => setQuoteProduct(p[lang].name)}
                  className="w-full py-3 bg-slate-900 hover:bg-slate-800 text-white rounded-xl font-medium transition-colors"
                >
                  {t.products.reqQuote}
                </button>
              </div>
            </div>
          )) : (
            <div className="col-span-full text-center py-20 text-slate-500">{t.products.noResults}</div>
          )}
        </div>
      </div>
    );
  };

  const QuotationModal = () => {
    const [loading, setLoading] = useState(false);
    const [formData, setFormData] = useState({
      name: '', contactPerson: '', email: '', phone: '', country: '',
      product: quoteProduct === 'General' ? '' : (quoteProduct || ''), qty: '', unit: '', dest: '', notes: ''
    });

    if (!quoteProduct) return null;

    const handleSubmit = async (e) => {
      e.preventDefault();
      if (!user || !db) {
        showToast("Database not initialized.");
        return;
      }
      setLoading(true);
      try {
        const docRef = await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'enquiries'), {
          ...formData,
          status: 'New',
          createdAt: serverTimestamp(),
          language: lang
        });
        showToast(t.form.success + docRef.id.slice(0,6).toUpperCase());
        setQuoteProduct(null);
      } catch (err) {
        console.error("Error submitting form:", err);
        showToast("Error submitting enquiry.");
      }
      setLoading(false);
    };

    return (
      <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm animate-in fade-in">
        <div className="bg-white w-full max-w-2xl rounded-2xl shadow-2xl overflow-hidden max-h-[90vh] flex flex-col">
          <div className="px-6 py-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
            <h3 className="text-xl font-bold text-slate-900">{t.form.title}</h3>
            <button onClick={() => setQuoteProduct(null)} className="text-slate-400 hover:text-slate-900 bg-white rounded-full p-1"><X className="w-5 h-5"/></button>
          </div>
          <div className="p-6 overflow-y-auto flex-1">
            <form onSubmit={handleSubmit} className="space-y-4">
              <div className="grid md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.name}</label>
                  <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.name} onChange={e=>setFormData({...formData, name: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.contactPerson}</label>
                  <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.contactPerson} onChange={e=>setFormData({...formData, contactPerson: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.email}</label>
                  <input required type="email" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.email} onChange={e=>setFormData({...formData, email: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.phone}</label>
                  <input required type="tel" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.phone} onChange={e=>setFormData({...formData, phone: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.country}</label>
                  <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.country} onChange={e=>setFormData({...formData, country: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.product}</label>
                  <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none bg-slate-50" value={formData.product} readOnly={quoteProduct !== 'General'} onChange={e=>setFormData({...formData, product: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.qty}</label>
                  <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.qty} onChange={e=>setFormData({...formData, qty: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.unit}</label>
                  <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.unit} onChange={e=>setFormData({...formData, unit: e.target.value})} />
                </div>
              </div>
              <div>
                <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.dest}</label>
                <input required type="text" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.dest} onChange={e=>setFormData({...formData, dest: e.target.value})} />
              </div>
              <div>
                <label className="block text-sm font-medium text-slate-700 mb-1">{t.form.notes}</label>
                <textarea rows="3" className="w-full p-3 border border-slate-200 rounded-xl focus:ring-2 focus:ring-orange-500 outline-none" value={formData.notes} onChange={e=>setFormData({...formData, notes: e.target.value})}></textarea>
              </div>
              <div className="pt-4 border-t border-slate-100">
                <button disabled={loading} type="submit" className="w-full py-4 bg-orange-500 hover:bg-orange-600 disabled:bg-orange-300 text-white font-bold rounded-xl shadow-lg transition-all flex justify-center items-center gap-2">
                  {loading ? t.form.sending : t.form.submit}
                </button>
                <p className="text-center text-xs text-slate-500 mt-3">Notifications (Email/WhatsApp) will be dispatched to INDRA administrators upon submission.</p>
              </div>
            </form>
          </div>
        </div>
      </div>
    );
  };

  const AdminDashboard = () => {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [password, setPassword] = useState('');
    const [enquiries, setEnquiries] = useState([]);

    useEffect(() => {
      if (!isAuthenticated || !user || !db) return;
      const q = collection(db, 'artifacts', appId, 'public', 'data', 'enquiries');
      const unsubscribe = onSnapshot(q, (snapshot) => {
        const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        // Sort descending locally to avoid complex Firestore indices rules
        data.sort((a, b) => (b.createdAt?.toMillis() || 0) - (a.createdAt?.toMillis() || 0));
        setEnquiries(data);
      }, (error) => {
        console.error("Error fetching enquiries: ", error);
        showToast("Error loading dashboard data.");
      });
      return () => unsubscribe();
    }, [isAuthenticated, user, db]);

    const handleLogin = (e) => {
      e.preventDefault();
      if (password === 'indra2026') setIsAuthenticated(true);
      else showToast(t.admin.error);
    };

    const updateStatus = async (id, newStatus) => {
      try {
        await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'enquiries', id), { status: newStatus });
        showToast(`Status updated to ${newStatus}`);
      } catch (err) {
        showToast("Update failed");
      }
    };

    if (!isAuthenticated) {
      return (
        <div className="min-h-[70vh] flex items-center justify-center p-4 bg-slate-50">
          <form onSubmit={handleLogin} className="bg-white p-8 rounded-2xl shadow-xl w-full max-w-sm border border-slate-100 text-center">
            <div className="w-16 h-16 bg-slate-900 rounded-full flex items-center justify-center mx-auto mb-6">
              <ShieldCheck className="text-orange-500 w-8 h-8" />
            </div>
            <h2 className="text-2xl font-bold text-slate-900 mb-6">{t.admin.login}</h2>
            <input type="password" placeholder={t.admin.passPlaceholder} value={password} onChange={e=>setPassword(e.target.value)} className="w-full p-3 border border-slate-200 rounded-xl mb-4 focus:ring-2 focus:ring-slate-900 outline-none text-center" />
            <button type="submit" className="w-full py-3 bg-slate-900 hover:bg-slate-800 text-white rounded-xl font-bold">Login</button>
          </form>
        </div>
      );
    }

    return (
      <div className="p-4 md:p-8 max-w-7xl mx-auto min-h-screen">
        <h1 className="text-3xl font-bold text-slate-900 mb-8">{t.admin.title}</h1>
        <div className="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
          <div className="overflow-x-auto">
            <table className="w-full text-left border-collapse">
              <thead>
                <tr className="bg-slate-50 border-b border-slate-200">
                  <th className="p-4 font-semibold text-slate-600 text-sm">Date</th>
                  <th className="p-4 font-semibold text-slate-600 text-sm">Company/Buyer</th>
                  <th className="p-4 font-semibold text-slate-600 text-sm">Product & Qty</th>
                  <th className="p-4 font-semibold text-slate-600 text-sm">Destination</th>
                  <th className="p-4 font-semibold text-slate-600 text-sm">Status</th>
                  <th className="p-4 font-semibold text-slate-600 text-sm">Actions</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-100">
                {enquiries.map(enq => (
                  <tr key={enq.id} className="hover:bg-slate-50/50">
                    <td className="p-4 text-sm text-slate-500 whitespace-nowrap">
                      {enq.createdAt?.toDate ? enq.createdAt.toDate().toLocaleDateString() : 'Just now'}
                    </td>
                    <td className="p-4">
                      <div className="font-bold text-slate-900">{enq.name}</div>
                      <div className="text-xs text-slate-500">{enq.email} <br/> {enq.phone}</div>
                    </td>
                    <td className="p-4 text-sm">
                      <span className="font-medium text-slate-900">{enq.product}</span><br/>
                      {enq.qty} {enq.unit}
                    </td>
                    <td className="p-4 text-sm text-slate-600">{enq.dest}, {enq.country}</td>
                    <td className="p-4">
                      <span className={`inline-flex px-2 py-1 text-xs font-bold rounded-full ${
                        enq.status === 'New' ? 'bg-blue-100 text-blue-700' :
                        enq.status === 'Contacted' ? 'bg-orange-100 text-orange-700' : 'bg-green-100 text-green-700'
                      }`}>
                        {enq.status}
                      </span>
                    </td>
                    <td className="p-4 text-sm flex gap-2 flex-wrap max-w-[200px]">
                       <button onClick={() => updateStatus(enq.id, 'Contacted')} className="text-xs bg-slate-100 hover:bg-slate-200 px-2 py-1 rounded text-slate-700 border border-slate-200">Contacted</button>
                       <button onClick={() => updateStatus(enq.id, 'Confirmed')} className="text-xs bg-green-50 hover:bg-green-100 px-2 py-1 rounded text-green-700 border border-green-200">Confirmed</button>
                    </td>
                  </tr>
                ))}
                {enquiries.length === 0 && (
                  <tr><td colSpan="6" className="p-8 text-center text-slate-500">No enquiries found.</td></tr>
                )}
              </tbody>
            </table>
          </div>
        </div>
      </div>
    );
  };

  return (
    <div className="min-h-screen bg-white font-sans text-slate-900 selection:bg-orange-500 selection:text-white">
      <Navbar />
      
      <main>
        {view === 'home' && <HomeView />}
        {view === 'products' && <ProductsView />}
        {view === 'about' && (
          <div className="py-20 max-w-4xl mx-auto px-4 text-center animate-in fade-in">
             <div className="w-20 h-20 bg-slate-900 rounded-full mx-auto flex items-center justify-center mb-8">
               <Globe className="text-orange-500 w-10 h-10" />
             </div>
             <h1 className="text-4xl font-bold mb-8">{t.about.title}</h1>
             <p className="text-lg text-slate-600 leading-relaxed mb-6">{t.about.p1}</p>
             <p className="text-lg text-slate-600 leading-relaxed">{t.about.p2}</p>
          </div>
        )}
        {view === 'contact' && (
           <div className="py-20 max-w-3xl mx-auto px-4 text-center animate-in fade-in">
             <h1 className="text-4xl font-bold mb-12">{t.contact.title}</h1>
             <div className="grid sm:grid-cols-3 gap-8">
                <div className="bg-slate-50 p-6 rounded-2xl">
                  <MapPin className="w-8 h-8 text-orange-500 mx-auto mb-4" />
                  <p className="font-medium text-slate-900">{t.contact.address}</p>
                </div>
                <div className="bg-slate-50 p-6 rounded-2xl">
                  <Mail className="w-8 h-8 text-orange-500 mx-auto mb-4" />
                  <p className="font-medium text-slate-900">{t.contact.email}</p>
                </div>
                <div className="bg-slate-50 p-6 rounded-2xl">
                  <Phone className="w-8 h-8 text-orange-500 mx-auto mb-4" />
                  <p className="font-medium text-slate-900">{t.contact.phone}</p>
                </div>
             </div>
           </div>
        )}
        {view === 'admin' && <AdminDashboard />}
      </main>

      <footer className="bg-slate-900 text-slate-400 py-8 border-t border-slate-800 mt-auto">
        <div className="max-w-7xl mx-auto px-4 text-center text-sm">
          {t.footer.text}
        </div>
      </footer>

      <QuotationModal />
      <Toast />
    </div>
  );
}
