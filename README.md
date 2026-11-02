# NMOBI[nmobi.tsx](https://github.com/user-attachments/files/34261628/nmobi.tsx)
import React, { useState, useEffect, useMemo } from 'react';
import { initializeApp } from 'firebase/app';
import {
  getAuth,
  signInAnonymously,
  signInWithCustomToken,
  onAuthStateChanged,
} from 'firebase/auth';
import {
  getFirestore,
  collection,
  doc,
  setDoc,
  addDoc,
  updateDoc,
  deleteDoc,
  onSnapshot,
} from 'firebase/firestore';
import {
  Home,
  Info,
  ShoppingBag,
  Truck,
  Phone,
  Search,
  Sparkles,
  ShieldCheck,
  HeartHandshake,
  Compass,
  MapPin,
  MessageCircle,
  PhoneCall,
  ChevronRight,
  Plus,
  Minus,
  X,
  CheckCircle2,
  ExternalLink,
  PackageOpen,
  SlidersHorizontal,
  Lock,
  LogOut,
  Package,
  FolderKanban,
  ShoppingCart,
  BarChart3,
  Trash2,
  Edit3,
  Eye,
  EyeOff,
  Upload,
  AlertCircle,
  ArrowLeft,
  Check,
  Database,
  KeyRound,
} from 'lucide-react';

// ============================================================================
// FIREBASE CLOUD INITIALIZATION
// ============================================================================
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'nmobi-bishkek-store';

// ============================================================================
// DEFAULT CATEGORIES & OPTIONAL STARTER PRODUCTS FOR SEEDING
// ============================================================================
const DEFAULT_CATEGORIES = [
  { id: 'cat-chargers', name: 'Зарядные устройства', sortOrder: 1 },
  { id: 'cat-cables', name: 'Кабели', sortOrder: 2 },
  { id: 'cat-headphones', name: 'Наушники', sortOrder: 3 },
  { id: 'cat-cases', name: 'Чехлы', sortOrder: 4 },
  { id: 'cat-glass', name: 'Защитные стёкла', sortOrder: 5 },
  { id: 'cat-powerbanks', name: 'Power Bank', sortOrder: 6 },
  { id: 'cat-car', name: 'Автомобильные аксессуары', sortOrder: 7 },
  { id: 'cat-smart', name: 'Умные гаджеты', sortOrder: 8 },
  { id: 'cat-other', name: 'Другие аксессуары', sortOrder: 9 },
];

const STARTER_PRODUCTS = [
  {
    name: 'GaN Зарядное устройство 45W Dual Type-C',
    categoryId: 'cat-chargers',
    price: 1850,
    shortDescription: 'Компактный сетевой адаптер с поддержкой быстрой зарядки PD 3.0 и PPS для iPhone и Samsung.',
    fullDescription: 'Премиальный GaN-адаптер мощностью 45 Вт с двумя портами USB-C. Обеспечивает безопасную сверхбыструю зарядку смартфонов, планшетов и ультрабуков без нагрева корпуса. Идеально подходит для iPhone 16 Pro Max и Samsung Galaxy.',
    imageUrl: 'https://images.unsplash.com/photo-1583863788434-e58a36330cf0?auto=format&fit=crop&w=900&q=80',
    inStock: true,
    isVisible: true,
    ordersCount: 14,
  },
  {
    name: 'Кабель в тканевой оплётке Type-C — Type-C 100W (1.5 м)',
    categoryId: 'cat-cables',
    price: 750,
    shortDescription: 'Усиленный нейлоновый кабель с интеллектуальным чипом E-Marker и защитой от перегиба.',
    fullDescription: 'Износостойкий кабель премиум-класса в плотной осенне-графитовой оплётке. Поддерживает мощность до 100 Вт и быструю передачу данных. Коннекторы из анодированного алюминия выдерживают более 25 000 сгибаний.',
    imageUrl: 'https://images.unsplash.com/photo-1544716278-ca5e3f4abd8c?auto=format&fit=crop&w=900&q=80',
    inStock: true,
    isVisible: true,
    ordersCount: 22,
  },
  {
    name: 'Беспроводные наушники ANC Studio Pods Pro',
    categoryId: 'cat-headphones',
    price: 3490,
    shortDescription: 'Глубокое объёмное звучание, активное шумоподавление и до 30 часов автономной работы.',
    fullDescription: 'Эргономичные беспроводные наушники с режимом прозрачности и гибридным активным шумоподавлением (ANC). Мягкие амбушюры, мгновенное подключение по Bluetooth 5.4 и поддержка беспроводной зарядки кейса.',
    imageUrl: 'https://images.unsplash.com/photo-1590658268037-6bf12165a8df?auto=format&fit=crop&w=900&q=80',
    inStock: true,
    isVisible: true,
    ordersCount: 19,
  },
  {
    name: 'Ультратонкий чехол Aramid MagSafe Touch',
    categoryId: 'cat-cases',
    price: 1490,
    shortDescription: 'Тактильное матовое покрытие, встроенное магнитное кольцо MagSafe и защита камер.',
    fullDescription: 'Стильный минималистичный чехол с бархатистым покрытием Soft-Touch и усиленной алюминиевой рамкой вокруг блока камер. Мощные неодимовые магниты обеспечивают надёжную фиксацию аксессуаров и зарядных устройств.',
    imageUrl: 'https://images.unsplash.com/photo-1601593346740-925612772716?auto=format&fit=crop&w=900&q=80',
    inStock: true,
    isVisible: true,
    ordersCount: 17,
  },
  {
    name: 'Магнитный Power Bank 10 000 mAh UltraSlim',
    categoryId: 'cat-powerbanks',
    price: 2650,
    shortDescription: 'Беспроводной внешний аккумулятор в алюминиевом корпусе с поддержкой MagSafe 15W и PD 20W.',
    fullDescription: 'Невероятно тонкий внешний аккумулятор толщиной всего 12 мм. Надёжно крепится к спинке смартфона и заряжает его без проводов. Оснащён индикатором заряда и защитой от перегрева.',
    imageUrl: 'https://images.unsplash.com/photo-1609091839311-d5365f9ff1c5?auto=format&fit=crop&w=900&q=80',
    inStock: true,
    isVisible: true,
    ordersCount: 11,
  },
  {
    name: 'Защитное сапфировое стекло 3D Full Fit с антибликом',
    categoryId: 'cat-glass',
    price: 690,
    shortDescription: 'Идеальная прозрачность, олеофобное покрытие премиум-класса и рамка для лёгкой установки.',
    fullDescription: 'Закалённое стекло повышенной прочности 9H со скруглёнными краями. Не искажает цветопередачу дисплея и защищает экран при падениях. В комплекте установочная рамка и набор для очистки.',
    imageUrl: 'https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?auto=format&fit=crop&w=900&q=80',
    inStock: true,
    isVisible: true,
    ordersCount: 28,
  },
];

// ============================================================================
// UTILITY: CLIENT-SIDE IMAGE RESIZER FOR PRODUCT UPLOADS
// ============================================================================
function compressImageFile(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (event) => {
      const img = new Image();
      img.onload = () => {
        const canvas = document.createElement('canvas');
        const maxDim = 750;
        let width = img.width;
        let height = img.height;

        if (width > height && width > maxDim) {
          height = Math.round((height * maxDim) / width);
          width = maxDim;
        } else if (height > maxDim) {
          width = Math.round((width * maxDim) / height);
          height = maxDim;
        }

        canvas.width = width;
        canvas.height = height;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0, width, height);
        const dataUrl = canvas.toDataURL('image/jpeg', 0.82);
        resolve(dataUrl);
      };
      img.onerror = reject;
      img.src = event.target.result;
    };
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

// ============================================================================
// UTILITY: SHA-256 PASSWORD HASHING FOR SERVER VERIFICATION
// ============================================================================
async function hashSecretPassword(password) {
  const encoder = new TextEncoder();
  const data = encoder.encode('nmobi_salt_2026_' + password.trim());
  const hashBuffer = await crypto.subtle.digest('SHA-256', data);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map((b) => b.toString(16).padStart(2, '0')).join('');
}

// ============================================================================
// MAIN APPLICATION COMPONENT
// ============================================================================
export default function App() {
  // 1. Firebase Auth & Cloud State
  const [user, setUser] = useState(null);
  const [dbLoading, setDbLoading] = useState(true);
  const [categories, setCategories] = useState([]);
  const [products, setProducts] = useState([]);
  const [orders, setOrders] = useState([]);
  const [adminConfig, setAdminConfig] = useState(null);
  const [adminSessionVerified, setAdminSessionVerified] = useState(false);

  // 2. Routing State (Public Storefront vs Hidden /admin Route)
  const [isRouteAdmin, setIsRouteAdmin] = useState(false);
  const [activeTab, setActiveTab] = useState('home');

  // 3. Public Catalog Filters & Cart
  const [selectedCategoryId, setSelectedCategoryId] = useState('all');
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedProductModal, setSelectedProductModal] = useState(null);
  const [cart, setCart] = useState([]);
  const [isCartOpen, setIsCartOpen] = useState(false);

  // 4. Checkout Form State
  const [customerName, setCustomerName] = useState('');
  const [customerPhone, setCustomerPhone] = useState('');
  const [deliveryAddress, setDeliveryAddress] = useState('');
  const [orderComment, setOrderComment] = useState('');
  const [submittingOrder, setSubmittingOrder] = useState(false);
  const [orderSuccessData, setOrderSuccessData] = useState(null);
  const [orderFormError, setOrderFormError] = useState('');

  // 5. Admin Panel State
  const [adminSection, setAdminSection] = useState('stats');
  const [adminPasswordInput, setAdminPasswordInput] = useState('');
  const [adminAuthError, setAdminAuthError] = useState('');
  const [adminAuthLoading, setAdminAuthLoading] = useState(false);
  const [adminToast, setAdminToast] = useState(null);

  // Admin Product Form
  const [editingProduct, setEditingProduct] = useState(null);
  const [prodName, setProdName] = useState('');
  const [prodCategoryId, setProdCategoryId] = useState('cat-chargers');
  const [prodPrice, setProdPrice] = useState('');
  const [prodShortDesc, setProdShortDesc] = useState('');
  const [prodFullDesc, setProdFullDesc] = useState('');
  const [prodImageUrl, setProdImageUrl] = useState('');
  const [prodInStock, setProdInStock] = useState(true);
  const [prodIsVisible, setProdIsVisible] = useState(true);
  const [uploadingPhoto, setUploadingPhoto] = useState(false);
  const [deleteConfirmId, setDeleteConfirmId] = useState(null);

  // Admin Category Form
  const [newCategoryName, setNewCategoryName] = useState('');
  const [editingCategoryId, setEditingCategoryId] = useState(null);
  const [editingCategoryName, setEditingCategoryName] = useState('');

  const showToast = (type, text) => {
    setAdminToast({ type, text });
    setTimeout(() => setAdminToast(null), 4000);
  };

  // ==========================================================================
  // URL / HASH ROUTING & SECRET KEYBOARD / SEARCH ENTRY TO /admin
  // ==========================================================================
  useEffect(() => {
    const checkRoute = () => {
      const hash = window.location.hash.toLowerCase();
      const pathname = window.location.pathname.toLowerCase();
      if (hash === '#/admin' || hash === '#admin' || pathname.endsWith('/admin')) {
        setIsRouteAdmin(true);
      } else {
        setIsRouteAdmin(false);
      }
    };

    checkRoute();
    window.addEventListener('hashchange', checkRoute);
    window.addEventListener('popstate', checkRoute);

    // Keyboard shortcut Ctrl+Shift+A (or Cmd+Shift+A) to navigate to /admin
    const handleKeyDown = (e) => {
      if ((e.ctrlKey || e.metaKey) && e.shiftKey && e.key.toLowerCase() === 'a') {
        e.preventDefault();
        window.location.hash = '#/admin';
        setIsRouteAdmin(true);
      }
    };
    window.addEventListener('keydown', handleKeyDown);

    return () => {
      window.removeEventListener('hashchange', checkRoute);
      window.removeEventListener('popstate', checkRoute);
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, []);

  // ==========================================================================
  // FIREBASE AUTHENTICATION (RULE 3)
  // ==========================================================================
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) {
        console.error('Auth error:', err);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
    });
    return () => unsubscribe();
  }, []);

  // ==========================================================================
  // REAL-TIME FIRESTORE LISTENERS (RULE 1 & RULE 2)
  // ==========================================================================
  useEffect(() => {
    if (!user) return;

    const categoriesRef = collection(db, 'artifacts', appId, 'public', 'data', 'categories');
    const productsRef = collection(db, 'artifacts', appId, 'public', 'data', 'products');
    const ordersRef = collection(db, 'artifacts', appId, 'public', 'data', 'orders');
    const securityDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'security_config', 'admin_auth');

    const unsubCategories = onSnapshot(
      categoriesRef,
      async (snapshot) => {
        if (snapshot.empty) {
          // Seed default categories automatically if database is fresh
          for (const cat of DEFAULT_CATEGORIES) {
            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'categories', cat.id), {
              name: cat.name,
              sortOrder: cat.sortOrder,
              createdAt: new Date().toISOString(),
            });
          }
        } else {
          const list = snapshot.docs.map((d) => ({ id: d.id, ...d.data() }));
          list.sort((a, b) => (a.sortOrder || 99) - (b.sortOrder || 99));
          setCategories(list);
        }
      },
      (err) => console.error('Categories listener error:', err)
    );

    const unsubProducts = onSnapshot(
      productsRef,
      (snapshot) => {
        const list = snapshot.docs.map((d) => ({ id: d.id, ...d.data() }));
        list.sort((a, b) => (b.createdAt || '').localeCompare(a.createdAt || ''));
        setProducts(list);
        setDbLoading(false);
      },
      (err) => {
        console.error('Products listener error:', err);
        setDbLoading(false);
      }
    );

    const unsubOrders = onSnapshot(
      ordersRef,
      (snapshot) => {
        const list = snapshot.docs.map((d) => ({ id: d.id, ...d.data() }));
        list.sort((a, b) => (b.createdAt || '').localeCompare(a.createdAt || ''));
        setOrders(list);
      },
      (err) => console.error('Orders listener error:', err)
    );

    const unsubSecurity = onSnapshot(
      securityDocRef,
      (docSnap) => {
        if (docSnap.exists()) {
          const data = docSnap.data();
          setAdminConfig(data);
          // Verify if current user uid is in authorizedSessionUids
          if (
            Array.isArray(data.authorizedSessionUids) &&
            data.authorizedSessionUids.includes(user.uid)
          ) {
            setAdminSessionVerified(true);
          }
        } else {
          setAdminConfig(null);
        }
      },
      (err) => console.error('Security config error:', err)
    );

    return () => {
      unsubCategories();
      unsubProducts();
      unsubOrders();
      unsubSecurity();
    };
  }, [user]);

  // ==========================================================================
  // PUBLIC CATALOG COMPUTED DATA
  // ==========================================================================
  const categoryMap = useMemo(() => {
    const map = {};
    categories.forEach((c) => {
      map[c.id] = c.name;
    });
    return map;
  }, [categories]);

  const publicVisibleProducts = useMemo(() => {
    return products.filter((p) => p.isVisible !== false);
  }, [products]);

  const filteredPublicProducts = useMemo(() => {
    return publicVisibleProducts.filter((p) => {
      const matchesCat =
        selectedCategoryId === 'all' || p.categoryId === selectedCategoryId;
      const q = searchQuery.toLowerCase().trim();
      const matchesSearch =
        !q ||
        (p.name && p.name.toLowerCase().includes(q)) ||
        (p.shortDescription && p.shortDescription.toLowerCase().includes(q));
      return matchesCat && matchesSearch;
    });
  }, [publicVisibleProducts, selectedCategoryId, searchQuery]);

  // Handle typing "/admin" inside search bar to open /admin route seamlessly
  const handleSearchChange = (val) => {
    if (val.trim().toLowerCase() === '/admin') {
      setSearchQuery('');
      window.location.hash = '#/admin';
      setIsRouteAdmin(true);
      return;
    }
    setSearchQuery(val);
  };

  // ==========================================================================
  // CART & CHECKOUT HANDLERS
  // ==========================================================================
  const addToCart = (product, openCartModal = false) => {
    if (!product.inStock) return;
    setCart((prev) => {
      const found = prev.find((item) => item.productId === product.id);
      if (found) {
        return prev.map((item) =>
          item.productId === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [
        ...prev,
        {
          productId: product.id,
          name: product.name,
          price: Number(product.price),
          quantity: 1,
          imageUrl: product.imageUrl,
        },
      ];
    });
    if (openCartModal) {
      setSelectedProductModal(null);
      setOrderSuccessData(null);
      setIsCartOpen(true);
    }
  };

  const updateCartQuantity = (productId, delta) => {
    setCart((prev) =>
      prev
        .map((item) =>
          item.productId === productId
            ? { ...item, quantity: item.quantity + delta }
            : item
        )
        .filter((item) => item.quantity > 0)
    );
  };

  const cartTotal = useMemo(
    () => cart.reduce((sum, item) => sum + item.price * item.quantity, 0),
    [cart]
  );

  const cartCount = useMemo(
    () => cart.reduce((sum, item) => sum + item.quantity, 0),
    [cart]
  );

  const handleCreateOrder = async (e) => {
    e.preventDefault();
    if (!user) return;
    setOrderFormError('');

    if (customerName.trim().length < 2) {
      setOrderFormError('Пожалуйста, укажите ваше имя (минимум 2 символа).');
      return;
    }
    if (customerPhone.trim().length < 6) {
      setOrderFormError('Пожалуйста, укажите корректный номер телефона.');
      return;
    }
    if (deliveryAddress.trim().length < 3) {
      setOrderFormError('Укажите адрес доставки по Бишкеку или «Самовывоз».');
      return;
    }
    if (cart.length === 0) {
      setOrderFormError('Корзина пуста.');
      return;
    }

    setSubmittingOrder(true);
    try {
      // Server-verified price calculation from live database products
      let verifiedSum = 0;
      const verifiedItems = cart.map((cItem) => {
        const liveProd = products.find((p) => p.id === cItem.productId);
        const unitPrice = liveProd ? Number(liveProd.price) : Number(cItem.price);
        verifiedSum += unitPrice * cItem.quantity;
        return {
          productId: cItem.productId,
          name: liveProd ? liveProd.name : cItem.name,
          price: unitPrice,
          quantity: cItem.quantity,
          imageUrl: liveProd ? liveProd.imageUrl : cItem.imageUrl,
        };
      });

      const orderDoc = {
        customerName: customerName.trim(),
        customerPhone: customerPhone.trim(),
        deliveryAddress: deliveryAddress.trim(),
        comment: orderComment.trim(),
        items: verifiedItems,
        totalAmount: verifiedSum,
        status: 'new',
        customerUid: user.uid,
        createdAt: new Date().toISOString(),
      };

      const ordersRef = collection(db, 'artifacts', appId, 'public', 'data', 'orders');
      const createdRef = await addDoc(ordersRef, orderDoc);

      // Increment ordersCount on ordered products for popularity analytics
      for (const item of verifiedItems) {
        const targetProd = products.find((p) => p.id === item.productId);
        if (targetProd) {
          const prodRef = doc(db, 'artifacts', appId, 'public', 'data', 'products', item.productId);
          await updateDoc(prodRef, {
            ordersCount: (Number(targetProd.ordersCount) || 0) + item.quantity,
          });
        }
      }

      setOrderSuccessData({ id: createdRef.id, totalAmount: verifiedSum });
      setCart([]);
      setCustomerName('');
      setCustomerPhone('');
      setDeliveryAddress('');
      setOrderComment('');
    } catch (err) {
      console.error('Order creation error:', err);
      setOrderFormError('Не удалось отправить заказ. Попробуйте ещё раз.');
    } finally {
      setSubmittingOrder(false);
    }
  };

  // ==========================================================================
  // ADMIN AUTHENTICATION & SERVER SESSION VERIFICATION
  // ==========================================================================
  const requireAdminGuard = () => {
    if (!user || !adminSessionVerified) {
      showToast('error', 'Отказано в доступе: требуется авторизация администратора');
      return false;
    }
    return true;
  };

  const handleAdminSetupOrLogin = async (e) => {
    e.preventDefault();
    if (!user) return;
    setAdminAuthError('');

    if (adminPasswordInput.trim().length < 4) {
      setAdminAuthError('Пароль должен содержать минимум 4 символа.');
      return;
    }

    setAdminAuthLoading(true);
    try {
      const hashedInput = await hashSecretPassword(adminPasswordInput);
      const securityDocRef = doc(
        db,
        'artifacts',
        appId,
        'public',
        'data',
        'security_config',
        'admin_auth'
      );

      // First-time initial setup if no admin password exists in Firestore yet
      if (!adminConfig || !adminConfig.passwordHash) {
        await setDoc(securityDocRef, {
          passwordHash: hashedInput,
          authorizedSessionUids: [user.uid],
          updatedAt: new Date().toISOString(),
        });
        setAdminSessionVerified(true);
        setAdminPasswordInput('');
        showToast('success', 'Пароль администратора установлен. Добро пожаловать в NMOBI CRM!');
      } else {
        // Verify password hash against cloud security config
        if (hashedInput === adminConfig.passwordHash) {
          const existingUids = Array.isArray(adminConfig.authorizedSessionUids)
            ? adminConfig.authorizedSessionUids
            : [];
          const updatedUids = Array.from(new Set([...existingUids, user.uid]));
          await updateDoc(securityDocRef, {
            authorizedSessionUids: updatedUids,
            lastLoginAt: new Date().toISOString(),
          });
          setAdminSessionVerified(true);
          setAdminPasswordInput('');
          showToast('success', 'Авторизация успешна!');
        } else {
          setAdminAuthError('Неверный пароль администратора.');
        }
      }
    } catch (err) {
      console.error('Admin auth error:', err);
      setAdminAuthError('Ошибка проверки прав доступа.');
    } finally {
      setAdminAuthLoading(false);
    }
  };

  const handleAdminLogout = async () => {
    if (user && adminConfig) {
      try {
        const securityDocRef = doc(
          db,
          'artifacts',
          appId,
          'public',
          'data',
          'security_config',
          'admin_auth'
        );
        const filtered = (adminConfig.authorizedSessionUids || []).filter(
          (uid) => uid !== user.uid
        );
        await updateDoc(securityDocRef, { authorizedSessionUids: filtered });
      } catch (e) {
        console.error(e);
      }
    }
    setAdminSessionVerified(false);
  };

  // ==========================================================================
  // ADMIN CRM ACTIONS (PRODUCTS, CATEGORIES, ORDERS)
  // ==========================================================================
  const handlePhotoFileChange = async (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    setUploadingPhoto(true);
    try {
      const compressedDataUrl = await compressImageFile(file);
      setProdImageUrl(compressedDataUrl);
      showToast('success', 'Фотография обработана и готова к сохранению');
    } catch {
      showToast('error', 'Не удалось обработать изображение');
    } finally {
      setUploadingPhoto(false);
    }
  };

  const resetProductForm = () => {
    setEditingProduct(null);
    setProdName('');
    setProdCategoryId(categories[0]?.id || 'cat-chargers');
    setProdPrice('');
    setProdShortDesc('');
    setProdFullDesc('');
    setProdImageUrl('');
    setProdInStock(true);
    setProdIsVisible(true);
  };

  const startEditProduct = (prod) => {
    setEditingProduct(prod);
    setProdName(prod.name || '');
    setProdCategoryId(prod.categoryId || categories[0]?.id || '');
    setProdPrice(String(prod.price ?? ''));
    setProdShortDesc(prod.shortDescription || '');
    setProdFullDesc(prod.fullDescription || '');
    setProdImageUrl(prod.imageUrl || '');
    setProdInStock(prod.inStock !== false);
    setProdIsVisible(prod.isVisible !== false);
  };

  const handleSaveProduct = async (e) => {
    e.preventDefault();
    if (!requireAdminGuard()) return;

    if (!prodName.trim() || !prodShortDesc.trim() || !prodImageUrl.trim()) {
      showToast('error', 'Заполните название, краткое описание и фотографию');
      return;
    }

    const numericPrice = Number(prodPrice);
    if (isNaN(numericPrice) || numericPrice < 0) {
      showToast('error', 'Укажите корректную цену в сомах');
      return;
    }

    try {
      const payload = {
        name: prodName.trim(),
        categoryId: prodCategoryId || (categories[0]?.id ?? 'cat-other'),
        price: numericPrice,
        shortDescription: prodShortDesc.trim(),
        fullDescription: prodFullDesc.trim(),
        imageUrl: prodImageUrl.trim(),
        inStock: Boolean(prodInStock),
        isVisible: Boolean(prodIsVisible),
        updatedAt: new Date().toISOString(),
      };

      if (editingProduct) {
        const prodRef = doc(
          db,
          'artifacts',
          appId,
          'public',
          'data',
          'products',
          editingProduct.id
        );
        await updateDoc(prodRef, payload);
        showToast('success', 'Товар обновлён в каталоге');
      } else {
        const productsRef = collection(
          db,
          'artifacts',
          appId,
          'public',
          'data',
          'products'
        );
        await addDoc(productsRef, {
          ...payload,
          ordersCount: 0,
          createdAt: new Date().toISOString(),
        });
        showToast('success', 'Новый товар добавлен в базу данных');
      }
      resetProductForm();
    } catch (err) {
      console.error(err);
      showToast('error', 'Ошибка при сохранении товара');
    }
  };

  const handleToggleProductVisibility = async (prod) => {
    if (!requireAdminGuard()) return;
    try {
      const prodRef = doc(db, 'artifacts', appId, 'public', 'data', 'products', prod.id);
      const nextState = prod.isVisible === false ? true : false;
      await updateDoc(prodRef, { isVisible: nextState });
      showToast(
        'success',
        nextState ? 'Товар возвращён в публичный каталог' : 'Товар скрыт из каталога'
      );
    } catch {
      showToast('error', 'Ошибка изменения видимости');
    }
  };

  const handleToggleProductStock = async (prod) => {
    if (!requireAdminGuard()) return;
    try {
      const prodRef = doc(db, 'artifacts', appId, 'public', 'data', 'products', prod.id);
      const nextStock = prod.inStock === false ? true : false;
      await updateDoc(prodRef, { inStock: nextStock });
      showToast('success', nextStock ? 'Статус: В наличии' : 'Статус: Нет в наличии');
    } catch {
      showToast('error', 'Ошибка обновления наличия');
    }
  };

  const handleConfirmDeleteProduct = async (productId) => {
    if (!requireAdminGuard()) return;
    try {
      await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'products', productId));
      setDeleteConfirmId(null);
      showToast('success', 'Товар удалён из базы данных');
    } catch {
      showToast('error', 'Ошибка удаления товара');
    }
  };

  const handleSeedStarterProducts = async () => {
    if (!requireAdminGuard()) return;
    try {
      const productsRef = collection(db, 'artifacts', appId, 'public', 'data', 'products');
      for (const item of STARTER_PRODUCTS) {
        await addDoc(productsRef, {
          ...item,
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString(),
        });
      }
      showToast('success', 'Демонстрационная коллекция товаров NMOBI загружена!');
    } catch {
      showToast('error', 'Не удалось загрузить стартовые товары');
    }
  };

  // Categories CRUD
  const handleAddCategory = async (e) => {
    e.preventDefault();
    if (!requireAdminGuard()) return;
    if (!newCategoryName.trim()) return;
    try {
      const categoriesRef = collection(
        db,
        'artifacts',
        appId,
        'public',
        'data',
        'categories'
      );
      await addDoc(categoriesRef, {
        name: newCategoryName.trim(),
        sortOrder: categories.length + 1,
        createdAt: new Date().toISOString(),
      });
      setNewCategoryName('');
      showToast('success', 'Категория добавлена');
    } catch {
      showToast('error', 'Ошибка добавления категории');
    }
  };

  const handleRenameCategory = async (catId) => {
    if (!requireAdminGuard()) return;
    if (!editingCategoryName.trim()) return;
    try {
      const catRef = doc(db, 'artifacts', appId, 'public', 'data', 'categories', catId);
      await updateDoc(catRef, { name: editingCategoryName.trim() });
      setEditingCategoryId(null);
      setEditingCategoryName('');
      showToast('success', 'Название категории обновлено');
    } catch {
      showToast('error', 'Ошибка переименования категории');
    }
  };

  const handleDeleteCategory = async (catId) => {
    if (!requireAdminGuard()) return;
    try {
      await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'categories', catId));
      showToast('success', 'Категория удалена');
    } catch {
      showToast('error', 'Ошибка удаления категории');
    }
  };

  // Orders Status Update
  const handleUpdateOrderStatus = async (orderId, newStatus) => {
    if (!requireAdminGuard()) return;
    try {
      const ordRef = doc(db, 'artifacts', appId, 'public', 'data', 'orders', orderId);
      await updateDoc(ordRef, { status: newStatus });
      showToast('success', 'Статус заказа обновлён');
    } catch {
      showToast('error', 'Не удалось обновить статус заказа');
    }
  };

  // Admin Statistics
  const totalOrdersSum = useMemo(() => {
    return orders
      .filter((o) => o.status !== 'cancelled')
      .reduce((acc, o) => acc + (Number(o.totalAmount) || 0), 0);
  }, [orders]);

  const popularProductsList = useMemo(() => {
    return [...products]
      .sort((a, b) => (Number(b.ordersCount) || 0) - (Number(a.ordersCount) || 0))
      .slice(0, 5);
  }, [products]);

  // ==========================================================================
  // RENDER 1: SEPARATE PROTECTED ADMIN PANEL (/admin)
  // ==========================================================================
  if (isRouteAdmin) {
    if (!adminSessionVerified) {
      const isInitialSetup = !adminConfig || !adminConfig.passwordHash;
      return (
        <div className="min-h-screen bg-[#252525] text-[#F4F0E8] flex items-center justify-center p-4">
          <div className="max-w-md w-full bg-[#F4F0E8] text-[#252525] rounded-3xl p-7 sm:p-9 shadow-2xl border border-[#E7DDCD] space-y-6">
            <div className="flex items-center justify-between">
              <div className="flex items-center gap-3">
                <div className="w-11 h-11 rounded-2xl bg-[#252525] text-[#C47B4A] flex items-center justify-center shadow-sm">
                  <Lock className="w-5 h-5" />
                </div>
                <div>
                  <h1 className="text-lg font-bold tracking-wider text-[#252525]">
                    NMOBI • CRM
                  </h1>
                  <p className="text-xs text-[#76543D]">
                    {isInitialSetup
                      ? 'Первоначальная настройка администратора'
                      : 'Защищённый вход по паролю'}
                  </p>
                </div>
              </div>
              <button
                onClick={() => {
                  window.location.hash = '';
                  setIsRouteAdmin(false);
                }}
                className="text-xs text-[#76543D] hover:text-[#252525] inline-flex items-center gap-1"
              >
                <ArrowLeft className="w-3.5 h-3.5" />
                <span>На сайт</span>
              </button>
            </div>

            {isInitialSetup && (
              <div className="p-3.5 rounded-2xl bg-[#E7DDCD]/70 border border-[#76543D]/20 text-xs text-[#76543D] leading-relaxed">
                <strong>Первая инициализация защиты:</strong> задайте мастер-пароль
                администратора. Он будет зашифрован (SHA-256) и сохранён в облачной
                конфигурации безопасности.
              </div>
            )}

            {adminAuthError && (
              <div className="p-3.5 rounded-2xl bg-red-100 text-red-800 text-xs font-medium flex items-center gap-2">
                <AlertCircle className="w-4 h-4 shrink-0" />
                <span>{adminAuthError}</span>
              </div>
            )}

            <form onSubmit={handleAdminSetupOrLogin} className="space-y-4">
              <div>
                <label className="text-xs font-semibold text-[#76543D] block mb-1.5">
                  {isInitialSetup
                    ? 'Создайте пароль администратора'
                    : 'Введите пароль администратора'}
                </label>
                <div className="relative">
                  <KeyRound className="w-4 h-4 text-[#76543D] absolute left-3.5 top-1/2 -translate-y-1/2" />
                  <input
                    type="password"
                    required
                    value={adminPasswordInput}
                    onChange={(e) => setAdminPasswordInput(e.target.value)}
                    placeholder="••••••••"
                    className="w-full pl-10 pr-4 py-3 rounded-xl bg-white border border-[#E7DDCD] text-sm text-[#252525] focus:outline-none focus:border-[#76543D]"
                  />
                </div>
              </div>

              <button
                type="submit"
                disabled={adminAuthLoading}
                className="w-full py-3.5 rounded-2xl bg-[#C47B4A] text-white font-semibold text-sm hover:opacity-95 transition-opacity shadow-md"
              >
                {adminAuthLoading
                  ? 'Проверка прав доступа...'
                  : isInitialSetup
                  ? 'Сохранить пароль и войти в CRM'
                  : 'Войти в панель управления'}
              </button>
            </form>
          </div>
        </div>
      );
    }

    // AUTHENTICATED ADMIN CRM VIEW
    return (
      <div className="min-h-screen bg-[#F4F0E8] text-[#252525] flex flex-col">
        {/* Top CRM Header */}
        <header className="bg-[#252525] text-[#F4F0E8] px-4 sm:px-8 py-4 flex flex-wrap items-center justify-between gap-3 sticky top-0 z-30 shadow-md">
          <div className="flex items-center gap-3">
            <span className="px-3 py-1 rounded-lg bg-[#C47B4A] text-white text-xs font-bold tracking-widest">
              NMOBI CRM
            </span>
            <span className="text-xs text-[#E7DDCD] hidden sm:inline">
              Защищённая система управления магазином (/admin)
            </span>
          </div>

          <div className="flex items-center gap-2.5">
            <button
              onClick={() => {
                window.location.hash = '';
                setIsRouteAdmin(false);
              }}
              className="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-white/10 text-xs font-medium hover:bg-white/20 transition-colors"
            >
              <ArrowLeft className="w-3.5 h-3.5" />
              <span>Витрина магазина</span>
            </button>
            <button
              onClick={handleAdminLogout}
              className="inline-flex items-center gap-1.5 px-3.5 py-2 rounded-xl bg-red-500/20 text-red-200 text-xs font-medium hover:bg-red-500/30 transition-colors"
            >
              <LogOut className="w-3.5 h-3.5" />
              <span>Выйти</span>
            </button>
          </div>
        </header>

        <div className="max-w-6xl w-full mx-auto px-4 py-6 space-y-6 flex-1">
          {/* Status Toast */}
          {adminToast && (
            <div
              className={`p-4 rounded-2xl text-xs font-semibold flex items-center justify-between shadow-sm ${
                adminToast.type === 'success'
                  ? 'bg-emerald-100 text-emerald-900 border border-emerald-200'
                  : 'bg-red-100 text-red-900 border border-red-200'
              }`}
            >
              <div className="flex items-center gap-2">
                <CheckCircle2 className="w-4 h-4 shrink-0" />
                <span>{adminToast.text}</span>
              </div>
              <button onClick={() => setAdminToast(null)}>
                <X className="w-4 h-4" />
              </button>
            </div>
          )}

          {/* CRM Navigation Tabs */}
          <div className="flex flex-wrap items-center justify-between gap-3">
            <div className="flex flex-wrap gap-2">
              {[
                { id: 'stats', label: 'Статистика', icon: BarChart3 },
                {
                  id: 'products',
                  label: `Управление товарами (${products.length})`,
                  icon: Package,
                },
                {
                  id: 'categories',
                  label: `Управление каталогом (${categories.length})`,
                  icon: FolderKanban,
                },
                {
                  id: 'orders',
                  label: `Заказы (${orders.length})`,
                  icon: ShoppingCart,
                },
              ].map((tab) => {
                const Icon = tab.icon;
                const active = adminSection === tab.id;
                return (
                  <button
                    key={tab.id}
                    onClick={() => setAdminSection(tab.id)}
                    className={`inline-flex items-center gap-2 px-4 py-2.5 rounded-2xl text-xs font-semibold transition-all ${
                      active
                        ? 'bg-[#252525] text-[#F4F0E8] shadow-sm'
                        : 'bg-white border border-[#E7DDCD] text-[#252525] hover:bg-[#E7DDCD]/60'
                    }`}
                  >
                    <Icon className="w-4 h-4 text-[#C47B4A]" />
                    <span>{tab.label}</span>
                  </button>
                );
              })}
            </div>

            {products.length === 0 && (
              <button
                onClick={handleSeedStarterProducts}
                className="inline-flex items-center gap-2 px-4 py-2.5 rounded-2xl bg-[#76543D] text-white text-xs font-semibold hover:opacity-95 transition-opacity"
              >
                <Database className="w-3.5 h-3.5" />
                <span>Загрузить стартовый набор товаров</span>
              </button>
            )}
          </div>

          {/* CRM TAB 1: СТАТИСТИКА */}
          {adminSection === 'stats' && (
            <div className="space-y-6">
              <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
                <div className="p-6 rounded-3xl bg-white border border-[#E7DDCD] shadow-sm">
                  <span className="text-xs font-medium text-[#76543D]">
                    Количество товаров в базе
                  </span>
                  <p className="text-3xl font-bold mt-1 text-[#252525]">
                    {products.length}
                  </p>
                  <span className="text-[11px] text-[#76543D]">
                    Активных в каталоге: {publicVisibleProducts.length}
                  </span>
                </div>

                <div className="p-6 rounded-3xl bg-white border border-[#E7DDCD] shadow-sm">
                  <span className="text-xs font-medium text-[#76543D]">
                    Количество заказов
                  </span>
                  <p className="text-3xl font-bold mt-1 text-[#252525]">
                    {orders.length}
                  </p>
                  <span className="text-[11px] text-[#76543D]">
                    Новых заказов: {orders.filter((o) => o.status === 'new').length}
                  </span>
                </div>

                <div className="p-6 rounded-3xl bg-[#252525] text-[#F4F0E8] shadow-sm">
                  <span className="text-xs text-[#E7DDCD]">Общая сумма заказов</span>
                  <p className="text-3xl font-bold mt-1 text-[#C47B4A]">
                    {totalOrdersSum.toLocaleString('ru-RU')} сом
                  </p>
                  <span className="text-[11px] text-[#E7DDCD]/70">
                    Без учёта отменённых заказов
                  </span>
                </div>
              </div>

              <div className="p-6 rounded-3xl bg-white border border-[#E7DDCD] shadow-sm space-y-4">
                <div className="flex items-center justify-between">
                  <h2 className="text-base font-bold text-[#252525]">
                    Популярные товары
                  </h2>
                  {products.length === 0 && (
                    <button
                      onClick={handleSeedStarterProducts}
                      className="text-xs text-[#C47B4A] font-semibold underline"
                    >
                      Заполнить витрину демо-товарами
                    </button>
                  )}
                </div>

                {popularProductsList.length === 0 ? (
                  <p className="text-xs text-[#76543D] py-4">
                    В каталоге пока нет товаров. Перейдите во вкладку «Управление
                    товарами», чтобы добавить первый товар.
                  </p>
                ) : (
                  <div className="divide-y divide-[#E7DDCD]">
                    {popularProductsList.map((p) => (
                      <div
                        key={p.id}
                        className="py-3 flex items-center justify-between gap-4 text-xs"
                      >
                        <div className="flex items-center gap-3">
                          <img
                            src={p.imageUrl}
                            alt={p.name}
                            className="w-11 h-11 rounded-xl object-cover bg-[#E7DDCD]"
                          />
                          <div>
                            <p className="font-bold text-[#252525]">{p.name}</p>
                            <p className="text-[#76543D]">
                              {Number(p.price).toLocaleString('ru-RU')} сом •{' '}
                              {categoryMap[p.categoryId] || 'Аксессуары'}
                            </p>
                          </div>
                        </div>
                        <span className="px-3 py-1 rounded-full bg-[#F4F0E8] border border-[#E7DDCD] font-semibold text-[#76543D]">
                          Заказов: {p.ordersCount || 0} шт.
                        </span>
                      </div>
                    ))}
                  </div>
                )}
              </div>
            </div>
          )}

          {/* CRM TAB 2: УПРАВЛЕНИЕ ТОВАРАМИ */}
          {adminSection === 'products' && (
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              {/* Left Form: Add / Edit Product */}
              <form
                onSubmit={handleSaveProduct}
                className="p-6 rounded-3xl bg-white border border-[#E7DDCD] shadow-sm space-y-4 h-fit"
              >
                <div className="flex items-center justify-between">
                  <h2 className="text-base font-bold text-[#252525]">
                    {editingProduct ? 'Редактировать товар' : 'Добавить новый товар'}
                  </h2>
                  {editingProduct && (
                    <button
                      type="button"
                      onClick={resetProductForm}
                      className="text-xs text-[#C47B4A] font-medium underline"
                    >
                      Сбросить
                    </button>
                  )}
                </div>

                <div className="space-y-1">
                  <label className="text-[11px] font-semibold text-[#76543D]">
                    Название товара *
                  </label>
                  <input
                    type="text"
                    required
                    value={prodName}
                    onChange={(e) => setProdName(e.target.value)}
                    placeholder="Например: Зарядный блок GaN 45W"
                    className="w-full px-3.5 py-2.5 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                  />
                </div>

                <div className="grid grid-cols-2 gap-2.5">
                  <div className="space-y-1">
                    <label className="text-[11px] font-semibold text-[#76543D]">
                      Цена (в сомах) *
                    </label>
                    <input
                      type="number"
                      required
                      min="0"
                      step="any"
                      value={prodPrice}
                      onChange={(e) => setProdPrice(e.target.value)}
                      placeholder="1850"
                      className="w-full px-3.5 py-2.5 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                    />
                  </div>

                  <div className="space-y-1">
                    <label className="text-[11px] font-semibold text-[#76543D]">
                      Категория *
                    </label>
                    <select
                      value={prodCategoryId}
                      onChange={(e) => setProdCategoryId(e.target.value)}
                      className="w-full px-3 py-2.5 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                    >
                      {categories.map((cat) => (
                        <option key={cat.id} value={cat.id}>
                          {cat.name}
                        </option>
                      ))}
                    </select>
                  </div>
                </div>

                <div className="space-y-1">
                  <label className="text-[11px] font-semibold text-[#76543D]">
                    Краткое описание *
                  </label>
                  <textarea
                    rows={2}
                    required
                    value={prodShortDesc}
                    onChange={(e) => setProdShortDesc(e.target.value)}
                    placeholder="Краткое описание для карточки каталога..."
                    className="w-full px-3.5 py-2 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                  />
                </div>

                <div className="space-y-1">
                  <label className="text-[11px] font-semibold text-[#76543D]">
                    Полное описание (в окне «Подробнее»)
                  </label>
                  <textarea
                    rows={3}
                    value={prodFullDesc}
                    onChange={(e) => setProdFullDesc(e.target.value)}
                    placeholder="Подробные характеристики, совместимость, комплектация..."
                    className="w-full px-3.5 py-2 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                  />
                </div>

                <div className="space-y-2">
                  <label className="text-[11px] font-semibold text-[#76543D] block">
                    Фотография товара *
                  </label>
                  <input
                    type="text"
                    required
                    value={prodImageUrl}
                    onChange={(e) => setProdImageUrl(e.target.value)}
                    placeholder="Вставьте ссылку на фото или загрузите файл ниже"
                    className="w-full px-3.5 py-2.5 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                  />
                  <div className="flex items-center gap-2">
                    <label className="inline-flex items-center gap-2 px-3.5 py-2 rounded-xl bg-[#E7DDCD] text-[#252525] text-xs font-medium cursor-pointer hover:bg-[#76543D] hover:text-white transition-colors">
                      <Upload className="w-3.5 h-3.5" />
                      <span>
                        {uploadingPhoto ? 'Обработка...' : 'Загрузить фото с устройства'}
                      </span>
                      <input
                        type="file"
                        accept="image/*"
                        onChange={handlePhotoFileChange}
                        className="hidden"
                      />
                    </label>
                    {prodImageUrl && (
                      <img
                        src={prodImageUrl}
                        alt="Предпросмотр"
                        className="w-9 h-9 rounded-lg object-cover border border-[#E7DDCD]"
                      />
                    )}
                  </div>
                </div>

                <div className="flex items-center justify-between pt-1 text-xs">
                  <label className="flex items-center gap-2 cursor-pointer">
                    <input
                      type="checkbox"
                      checked={prodInStock}
                      onChange={(e) => setProdInStock(e.target.checked)}
                    />
                    <span>В наличии</span>
                  </label>

                  <label className="flex items-center gap-2 cursor-pointer">
                    <input
                      type="checkbox"
                      checked={prodIsVisible}
                      onChange={(e) => setProdIsVisible(e.target.checked)}
                    />
                    <span>Показывать в каталоге</span>
                  </label>
                </div>

                <button
                  type="submit"
                  className="w-full py-3 rounded-xl bg-[#C47B4A] text-white text-xs font-semibold hover:opacity-95 transition-opacity shadow-sm"
                >
                  {editingProduct ? 'Сохранить изменения' : 'Добавить товар в каталог'}
                </button>
              </form>

              {/* Right Column: Products List */}
              <div className="lg:col-span-2 space-y-3">
                {products.length === 0 ? (
                  <div className="p-10 rounded-3xl bg-white border border-[#E7DDCD] text-center space-y-3">
                    <p className="text-sm font-semibold text-[#252525]">
                      В базе данных пока нет товаров
                    </p>
                    <p className="text-xs text-[#76543D]">
                      Добавьте товар через форму слева или загрузите демонстрационный
                      набор аксессуаров NMOBI.
                    </p>
                    <button
                      onClick={handleSeedStarterProducts}
                      className="px-5 py-2.5 rounded-xl bg-[#252525] text-[#F4F0E8] text-xs font-semibold"
                    >
                      Загрузить стартовые товары
                    </button>
                  </div>
                ) : (
                  products.map((p) => (
                    <div
                      key={p.id}
                      className="p-4 rounded-2xl bg-white border border-[#E7DDCD] flex flex-col sm:flex-row sm:items-center justify-between gap-4 shadow-sm"
                    >
                      <div className="flex items-center gap-3.5">
                        <img
                          src={p.imageUrl}
                          alt={p.name}
                          className="w-16 h-16 rounded-2xl object-cover bg-[#E7DDCD] shrink-0"
                        />
                        <div className="space-y-1">
                          <div className="flex items-center gap-2 flex-wrap">
                            <h3 className="text-sm font-bold text-[#252525]">
                              {p.name}
                            </h3>
                            {p.isVisible === false && (
                              <span className="px-2 py-0.5 rounded-md bg-neutral-200 text-neutral-700 text-[10px] font-semibold">
                                Скрыт из каталога
                              </span>
                            )}
                            {p.inStock === false && (
                              <span className="px-2 py-0.5 rounded-md bg-red-100 text-red-700 text-[10px] font-semibold">
                                Нет в наличии
                              </span>
                            )}
                          </div>
                          <p className="text-xs font-bold text-[#C47B4A]">
                            {Number(p.price).toLocaleString('ru-RU')} сом •{' '}
                            <span className="text-[#76543D] font-normal">
                              {categoryMap[p.categoryId] || 'Аксессуары'}
                            </span>
                          </p>
                          <p className="text-xs text-[#252525]/70 line-clamp-1">
                            {p.shortDescription}
                          </p>
                        </div>
                      </div>

                      <div className="flex items-center gap-2 shrink-0 flex-wrap">
                        <button
                          onClick={() => handleToggleProductStock(p)}
                          className="px-2.5 py-1.5 rounded-lg bg-[#F4F0E8] border border-[#E7DDCD] text-[11px] font-medium hover:bg-[#E7DDCD]"
                        >
                          {p.inStock !== false ? 'В наличии' : 'Нет в наличии'}
                        </button>

                        <button
                          onClick={() => handleToggleProductVisibility(p)}
                          title={
                            p.isVisible !== false
                              ? 'Скрыть товар из каталога'
                              : 'Вернуть товар в каталог'
                          }
                          className="p-2 rounded-lg bg-[#F4F0E8] border border-[#E7DDCD] hover:bg-[#E7DDCD]"
                        >
                          {p.isVisible !== false ? (
                            <Eye className="w-4 h-4 text-[#252525]" />
                          ) : (
                            <EyeOff className="w-4 h-4 text-[#76543D]" />
                          )}
                        </button>

                        <button
                          onClick={() => startEditProduct(p)}
                          title="Редактировать"
                          className="p-2 rounded-lg bg-[#F4F0E8] border border-[#E7DDCD] hover:bg-[#E7DDCD]"
                        >
                          <Edit3 className="w-4 h-4 text-[#252525]" />
                        </button>

                        {deleteConfirmId === p.id ? (
                          <div className="flex items-center gap-1">
                            <button
                              onClick={() => handleConfirmDeleteProduct(p.id)}
                              className="px-2.5 py-1.5 rounded-lg bg-red-600 text-white text-[11px] font-semibold"
                            >
                              Удалить
                            </button>
                            <button
                              onClick={() => setDeleteConfirmId(null)}
                              className="px-2 py-1.5 rounded-lg bg-neutral-200 text-neutral-700 text-[11px]"
                            >
                              Отмена
                            </button>
                          </div>
                        ) : (
                          <button
                            onClick={() => setDeleteConfirmId(p.id)}
                            title="Удалить товар"
                            className="p-2 rounded-lg bg-red-50 text-red-700 hover:bg-red-100"
                          >
                            <Trash2 className="w-4 h-4" />
                          </button>
                        )}
                      </div>
                    </div>
                  ))
                )}
              </div>
            </div>
          )}

          {/* CRM TAB 3: УПРАВЛЕНИЕ КАТЕГОРИЯМИ */}
          {adminSection === 'categories' && (
            <div className="max-w-2xl space-y-5">
              <form onSubmit={handleAddCategory} className="flex gap-2">
                <input
                  type="text"
                  required
                  value={newCategoryName}
                  onChange={(e) => setNewCategoryName(e.target.value)}
                  placeholder="Название новой категории..."
                  className="flex-1 px-4 py-3 rounded-2xl bg-white border border-[#E7DDCD] text-xs"
                />
                <button
                  type="submit"
                  className="inline-flex items-center gap-1.5 px-5 py-3 rounded-2xl bg-[#252525] text-[#F4F0E8] text-xs font-semibold hover:bg-[#76543D] transition-colors"
                >
                  <Plus className="w-4 h-4" />
                  <span>Добавить категорию</span>
                </button>
              </form>

              <div className="space-y-2.5">
                {categories.map((cat) => (
                  <div
                    key={cat.id}
                    className="p-4 rounded-2xl bg-white border border-[#E7DDCD] flex items-center justify-between gap-3"
                  >
                    {editingCategoryId === cat.id ? (
                      <div className="flex items-center gap-2 flex-1">
                        <input
                          type="text"
                          value={editingCategoryName}
                          onChange={(e) => setEditingCategoryName(e.target.value)}
                          className="flex-1 px-3 py-1.5 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs"
                        />
                        <button
                          onClick={() => handleRenameCategory(cat.id)}
                          className="px-3 py-1.5 rounded-xl bg-[#C47B4A] text-white text-xs font-medium inline-flex items-center gap-1"
                        >
                          <Check className="w-3.5 h-3.5" />
                          <span>Сохранить</span>
                        </button>
                      </div>
                    ) : (
                      <span className="text-sm font-semibold text-[#252525]">
                        {cat.name}
                      </span>
                    )}

                    <div className="flex items-center gap-2">
                      <button
                        onClick={() => {
                          setEditingCategoryId(cat.id);
                          setEditingCategoryName(cat.name);
                        }}
                        className="p-2 rounded-lg bg-[#F4F0E8] hover:bg-[#E7DDCD]"
                      >
                        <Edit3 className="w-3.5 h-3.5" />
                      </button>
                      <button
                        onClick={() => handleDeleteCategory(cat.id)}
                        className="p-2 rounded-lg bg-red-50 text-red-700 hover:bg-red-100"
                      >
                        <Trash2 className="w-3.5 h-3.5" />
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* CRM TAB 4: УПРАВЛЕНИЕ ЗАКАЗАМИ */}
          {adminSection === 'orders' && (
            <div className="space-y-4">
              {orders.length === 0 ? (
                <div className="p-10 rounded-3xl bg-white border border-[#E7DDCD] text-center text-xs text-[#76543D]">
                  Заказов пока не поступало. Когда покупатель оформит заказ на сайте,
                  он мгновенно появится здесь.
                </div>
              ) : (
                orders.map((ord) => (
                  <div
                    key={ord.id}
                    className="p-6 rounded-3xl bg-white border border-[#E7DDCD] space-y-4 shadow-sm"
                  >
                    <div className="flex flex-wrap items-start justify-between gap-4 border-b border-[#E7DDCD] pb-3">
                      <div className="space-y-1">
                        <span className="text-[11px] text-[#76543D]">
                          Дата:{' '}
                          {ord.createdAt
                            ? new Date(ord.createdAt).toLocaleString('ru-RU')
                            : 'Сегодня'}
                        </span>
                        <h3 className="text-base font-bold text-[#252525]">
                          {ord.customerName} •{' '}
                          <a
                            href={`tel:${ord.customerPhone}`}
                            className="text-[#C47B4A] underline"
                          >
                            {ord.customerPhone}
                          </a>
                        </h3>
                        <p className="text-xs text-[#76543D]">
                          Адрес доставки:{' '}
                          <strong className="text-[#252525]">
                            {ord.deliveryAddress}
                          </strong>
                        </p>
                        {ord.comment && (
                          <p className="text-xs text-[#76543D]">
                            Комментарий: {ord.comment}
                          </p>
                        )}
                      </div>

                      <div className="flex items-center gap-3">
                        <select
                          value={ord.status || 'new'}
                          onChange={(e) =>
                            handleUpdateOrderStatus(ord.id, e.target.value)
                          }
                          className="px-3.5 py-2 rounded-xl bg-[#F4F0E8] border border-[#E7DDCD] text-xs font-semibold"
                        >
                          <option value="new">Новый заказ</option>
                          <option value="processing">В обработке</option>
                          <option value="delivering">Доставляется (Яндекс)</option>
                          <option value="completed">Выполнен</option>
                          <option value="cancelled">Отменён</option>
                        </select>

                        <span className="text-base font-bold text-[#252525]">
                          {Number(ord.totalAmount).toLocaleString('ru-RU')} сом
                        </span>
                      </div>
                    </div>

                    <div className="grid grid-cols-1 sm:grid-cols-2 gap-2.5">
                      {(ord.items || []).map((item, idx) => (
                        <div
                          key={idx}
                          className="flex items-center gap-3 p-2.5 rounded-xl bg-[#F4F0E8] text-xs"
                        >
                          <img
                            src={item.imageUrl}
                            alt={item.name}
                            className="w-10 h-10 rounded-lg object-cover bg-[#E7DDCD]"
                          />
                          <div>
                            <p className="font-bold text-[#252525]">{item.name}</p>
                            <p className="text-[#76543D]">
                              {item.quantity} шт. ×{' '}
                              {Number(item.price).toLocaleString('ru-RU')} сом
                            </p>
                          </div>
                        </div>
                      ))}
                    </div>
                  </div>
                ))
              )}
            </div>
          )}
        </div>
      </div>
    );
  }

  // ==========================================================================
  // RENDER 2: PUBLIC NMOBI STOREFRONT (NO ADMIN LINKS ANYWHERE)
  // ==========================================================================
  return (
    <div className="min-h-screen flex flex-col bg-[#F4F0E8] text-[#252525] pb-28 selection:bg-[#C47B4A] selection:text-white">
      {/* TOP BRAND HEADER */}
      <header className="sticky top-0 z-30 backdrop-blur-md bg-[#F4F0E8]/90 border-b border-[#E7DDCD] transition-all">
        <div className="max-w-3xl mx-auto px-4 py-3.5 flex items-center justify-between">
          <button
            onClick={() => setActiveTab('home')}
            className="text-left group focus:outline-none"
          >
            <div className="flex items-center gap-2.5">
              <div className="w-9 h-9 rounded-xl bg-[#252525] text-[#F4F0E8] flex items-center justify-center font-bold tracking-widest text-xs shadow-sm group-hover:bg-[#76543D] transition-colors">
                NM
              </div>
              <div>
                <span className="text-xl font-bold tracking-[0.18em] text-[#252525] block leading-none">
                  NMOBI
                </span>
                <span className="text-[11px] text-[#76543D] tracking-wider">
                  Технологии, которые ближе
                </span>
              </div>
            </div>
          </button>

          <div className="flex items-center gap-2">
            <a
              href="https://wa.me/996704102050"
              target="_blank"
              rel="noopener noreferrer"
              className="hidden sm:inline-flex items-center gap-1.5 px-3.5 py-2 rounded-full text-xs font-medium bg-[#E7DDCD]/80 text-[#76543D] hover:bg-[#E7DDCD] transition-colors"
            >
              <MessageCircle className="w-3.5 h-3.5" />
              <span>WhatsApp</span>
            </a>

            <button
              onClick={() => {
                setOrderSuccessData(null);
                setIsCartOpen(true);
              }}
              className="relative inline-flex items-center gap-2 px-4 py-2 rounded-full bg-[#252525] text-[#F4F0E8] text-xs font-medium hover:bg-[#76543D] transition-all shadow-sm"
            >
              <ShoppingBag className="w-4 h-4 text-[#C47B4A]" />
              <span>Корзина</span>
              {cartCount > 0 && (
                <span className="px-1.5 py-0.5 text-[10px] rounded-full bg-[#C47B4A] text-white font-bold">
                  {cartCount}
                </span>
              )}
            </button>
          </div>
        </div>
      </header>

      {/* MAIN TAB CONTENT CONTAINER */}
      <main className="flex-1 max-w-3xl w-full mx-auto px-4 pt-6 space-y-8">
        {/* =============================================================== */}
        {/* TAB 1: ГЛАВНАЯ СТРАНИЦА                                         */}
        {/* =============================================================== */}
        {activeTab === 'home' && (
          <div className="space-y-8">
            {/* Hero Banner */}
            <section className="relative overflow-hidden rounded-3xl bg-[#252525] text-[#F4F0E8] p-6 sm:p-10 shadow-xl">
              <div className="relative z-10 max-w-xl space-y-5">
                <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-white/10 text-[#E7DDCD] text-xs tracking-wide">
                  <Sparkles className="w-3.5 h-3.5 text-[#C47B4A]" />
                  <span>Modern Premium • Бишкек</span>
                </div>

                <div className="space-y-2">
                  <h1 className="text-3xl sm:text-5xl font-bold tracking-[0.14em]">
                    NMOBI
                  </h1>
                  <p className="text-lg sm:text-xl font-medium text-[#C47B4A]">
                    «Технологии, которые ближе»
                  </p>
                </div>

                <p className="text-sm sm:text-base text-[#E7DDCD]/90 leading-relaxed">
                  NMOBI — пространство современных мобильных аксессуаров, где
                  технологии встречаются с эстетикой. Мы выбираем аксессуары,
                  которые делают повседневную жизнь комфортнее, защищённее и
                  стильнее.
                </p>

                <div className="pt-2 flex flex-wrap gap-3">
                  <button
                    onClick={() => setActiveTab('catalog')}
                    className="inline-flex items-center gap-2 px-6 py-3.5 rounded-2xl bg-[#C47B4A] text-white font-medium text-sm hover:opacity-95 transition-all shadow-md"
                  >
                    <span>Перейти в каталог</span>
                    <ChevronRight className="w-4 h-4" />
                  </button>
                  <button
                    onClick={() => setActiveTab('contacts')}
                    className="inline-flex items-center gap-2 px-6 py-3.5 rounded-2xl bg-white/10 text-[#F4F0E8] font-medium text-sm hover:bg-white/20 transition-all"
                  >
                    <span>Связаться с нами</span>
                  </button>
                </div>
              </div>

              <div className="absolute -right-16 -bottom-16 w-72 h-72 rounded-full bg-[#C47B4A]/20 blur-3xl pointer-events-none" />
            </section>

            {/* Главный баннер с изображениями смартфонов, зарядных устройств, наушников и аксессуаров */}
            <section className="space-y-3">
              <div className="flex items-center justify-between">
                <h2 className="text-xs font-semibold uppercase tracking-widest text-[#76543D]">
                  Коллекция направлений
                </h2>
                <button
                  onClick={() => setActiveTab('catalog')}
                  className="text-xs font-medium text-[#C47B4A] hover:underline"
                >
                  Открыть каталог →
                </button>
              </div>

              <div className="grid grid-cols-2 sm:grid-cols-4 gap-3">
                {[
                  {
                    title: 'Зарядные устройства',
                    subtitle: 'GaN адаптеры и MagSafe',
                    img: 'https://images.unsplash.com/photo-1583863788434-e58a36330cf0?auto=format&fit=crop&w=600&q=80',
                  },
                  {
                    title: 'Наушники и аудио',
                    subtitle: 'Чистое звучание без проводов',
                    img: 'https://images.unsplash.com/photo-1590658268037-6bf12165a8df?auto=format&fit=crop&w=600&q=80',
                  },
                  {
                    title: 'Чехлы и защита',
                    subtitle: 'Тактильный комфорт',
                    img: 'https://images.unsplash.com/photo-1601593346740-925612772716?auto=format&fit=crop&w=600&q=80',
                  },
                  {
                    title: 'Смарт-аксессуары',
                    subtitle: 'Эстетика и функциональность',
                    img: 'https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?auto=format&fit=crop&w=600&q=80',
                  },
                ].map((card, i) => (
                  <div
                    key={i}
                    onClick={() => setActiveTab('catalog')}
                    className="group cursor-pointer relative h-48 rounded-2xl overflow-hidden bg-[#E7DDCD] shadow-sm"
                  >
                    <img
                      src={card.img}
                      alt={card.title}
                      className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
                    />
                    <div className="absolute inset-0 bg-gradient-to-t from-[#252525]/90 via-[#252525]/30 to-transparent p-3.5 flex flex-col justify-end">
                      <p className="text-xs font-semibold text-[#F4F0E8]">
                        {card.title}
                      </p>
                      <p className="text-[11px] text-[#E7DDCD]/85">
                        {card.subtitle}
                      </p>
                    </div>
                  </div>
                ))}
              </div>
            </section>

            {/* Taplink-style Navigation Cards */}
            <section className="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
              <button
                onClick={() => setActiveTab('about')}
                className="flex items-center justify-between p-5 rounded-2xl bg-[#E7DDCD]/65 hover:bg-[#E7DDCD] transition-all text-left group"
              >
                <div className="space-y-1">
                  <span className="text-xs uppercase tracking-wider text-[#76543D] font-medium">
                    Философия и цели
                  </span>
                  <h3 className="text-base font-semibold text-[#252525]">
                    О магазине NMOBI
                  </h3>
                </div>
                <ChevronRight className="w-5 h-5 text-[#76543D] group-hover:translate-x-1 transition-transform" />
              </button>

              <button
                onClick={() => setActiveTab('delivery')}
                className="flex items-center justify-between p-5 rounded-2xl bg-[#E7DDCD]/65 hover:bg-[#E7DDCD] transition-all text-left group"
              >
                <div className="space-y-1">
                  <span className="text-xs uppercase tracking-wider text-[#76543D] font-medium">
                    г. Бишкек, пр. Чуй, 278
                  </span>
                  <h3 className="text-base font-semibold text-[#252525]">
                    Яндекс Доставка и наш адрес
                  </h3>
                </div>
                <ChevronRight className="w-5 h-5 text-[#76543D] group-hover:translate-x-1 transition-transform" />
              </button>
            </section>
          </div>
        )}

        {/* =============================================================== */}
        {/* TAB 2: О МАГАЗИНЕ + НАШИ ЦЕЛИ                                   */}
        {/* =============================================================== */}
        {activeTab === 'about' && (
          <div className="space-y-8">
            <section className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-6 sm:p-9 shadow-sm space-y-5">
              <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-[#E7DDCD] text-[#76543D] text-xs font-medium">
                <Info className="w-3.5 h-3.5" />
                <span>Философия бренда</span>
              </div>

              <h1 className="text-2xl sm:text-3xl font-bold text-[#252525]">
                О NMOBI
              </h1>

              <div className="space-y-4 text-sm sm:text-base text-[#252525]/90 leading-relaxed">
                <p>
                  Мы верим, что аксессуары — это не просто дополнение к
                  смартфону. Это часть вашего комфорта, стиля и повседневной
                  жизни.
                </p>
                <p>
                  NMOBI создан для тех, кто ценит качество, функциональность и
                  современный дизайн.
                </p>
                <p>
                  Наша задача — сделать выбор мобильных аксессуаров простым,
                  понятным и приятным.
                </p>
              </div>
            </section>

            {/* Вкладка «Наши цели» — 4 направления */}
            <section className="space-y-4">
              <div className="space-y-1">
                <span className="text-xs uppercase tracking-widest text-[#76543D] font-semibold">
                  Четыре направления
                </span>
                <h2 className="text-xl sm:text-2xl font-bold text-[#252525]">
                  Наши цели
                </h2>
              </div>

              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                {[
                  {
                    number: '01',
                    title: 'Качество',
                    desc: 'Предлагать аксессуары, которые соответствуют ожиданиям покупателей.',
                    icon: ShieldCheck,
                    badgeClass: 'bg-[#76543D] text-[#F4F0E8]',
                  },
                  {
                    number: '02',
                    title: 'Доступность',
                    desc: 'Делать современные технологии доступнее.',
                    icon: Compass,
                    badgeClass: 'bg-[#C47B4A] text-white',
                  },
                  {
                    number: '03',
                    title: 'Доверие',
                    desc: 'Строить долгосрочные отношения с клиентами.',
                    icon: HeartHandshake,
                    badgeClass: 'bg-[#252525] text-[#F4F0E8]',
                  },
                  {
                    number: '04',
                    title: 'Комфорт',
                    desc: 'Создавать удобный процесс покупки и обслуживания.',
                    icon: Sparkles,
                    badgeClass: 'bg-[#E7DDCD] text-[#76543D]',
                  },
                ].map((goal) => {
                  const IconComp = goal.icon;
                  return (
                    <div
                      key={goal.number}
                      className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-6 shadow-sm flex flex-col justify-between space-y-5 hover:border-[#76543D]/40 transition-all"
                    >
                      <div className="flex items-center justify-between">
                        <div
                          className={`w-11 h-11 rounded-2xl flex items-center justify-center ${goal.badgeClass} shadow-sm`}
                        >
                          <IconComp className="w-5 h-5" />
                        </div>
                        <span className="text-xs font-bold tracking-widest text-[#76543D]/50">
                          {goal.number}
                        </span>
                      </div>

                      <div className="space-y-1.5">
                        <h3 className="text-lg font-bold text-[#252525]">
                          {goal.title}
                        </h3>
                        <p className="text-sm text-[#252525]/80 leading-relaxed">
                          {goal.desc}
                        </p>
                      </div>
                    </div>
                  );
                })}
              </div>
            </section>
          </div>
        )}

        {/* =============================================================== */}
        {/* TAB 3: КАТАЛОГ ТОВАРОВ                                          */}
        {/* =============================================================== */}
        {activeTab === 'catalog' && (
          <div className="space-y-6">
            <div className="space-y-1.5">
              <h1 className="text-2xl sm:text-3xl font-bold text-[#252525]">
                Каталог товаров
              </h1>
              <p className="text-sm text-[#76543D]">
                Аксессуары с актуальными ценами и наличием из базы данных NMOBI
              </p>
            </div>

            {/* Search Input */}
            <div className="relative">
              <Search className="w-4 h-4 text-[#76543D] absolute left-4 top-1/2 -translate-y-1/2" />
              <input
                type="text"
                value={searchQuery}
                onChange={(e) => handleSearchChange(e.target.value)}
                placeholder="Поиск аксессуаров по названию или описанию..."
                className="w-full pl-11 pr-10 py-3.5 rounded-2xl bg-white/95 border border-[#E7DDCD] text-sm text-[#252525] placeholder:text-[#76543D]/60 focus:outline-none focus:border-[#76543D] shadow-sm"
              />
              {searchQuery && (
                <button
                  onClick={() => setSearchQuery('')}
                  className="absolute right-3.5 top-1/2 -translate-y-1/2 text-[#76543D] hover:text-[#252525]"
                >
                  <X className="w-4 h-4" />
                </button>
              )}
            </div>

            {/* Category Filter Pills */}
            <div className="flex items-center gap-2 overflow-x-auto pb-2">
              <button
                onClick={() => setSelectedCategoryId('all')}
                className={`shrink-0 px-4 py-2 rounded-full text-xs font-medium transition-all ${
                  selectedCategoryId === 'all'
                    ? 'bg-[#252525] text-[#F4F0E8] shadow-sm'
                    : 'bg-[#E7DDCD]/75 text-[#252525] hover:bg-[#E7DDCD]'
                }`}
              >
                Все категории
              </button>
              {categories.map((cat) => (
                <button
                  key={cat.id}
                  onClick={() => setSelectedCategoryId(cat.id)}
                  className={`shrink-0 px-4 py-2 rounded-full text-xs font-medium transition-all ${
                    selectedCategoryId === cat.id
                      ? 'bg-[#C47B4A] text-white shadow-sm'
                      : 'bg-[#E7DDCD]/75 text-[#252525] hover:bg-[#E7DDCD]'
                  }`}
                >
                  {cat.name}
                </button>
              ))}
            </div>

            {/* Product Grid or Empty State */}
            {dbLoading ? (
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                {[1, 2, 3, 4].map((n) => (
                  <div
                    key={n}
                    className="h-80 rounded-3xl bg-[#E7DDCD]/60 animate-pulse"
                  />
                ))}
              </div>
            ) : filteredPublicProducts.length === 0 ? (
              <div className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-12 text-center space-y-3 shadow-sm">
                <div className="w-12 h-12 rounded-2xl bg-[#E7DDCD] mx-auto flex items-center justify-center text-[#76543D]">
                  <PackageOpen className="w-6 h-6" />
                </div>
                <h3 className="text-lg font-semibold text-[#252525]">
                  Скоро здесь появятся товары
                </h3>
                <p className="text-xs text-[#76543D] max-w-sm mx-auto leading-relaxed">
                  Мы пополняем коллекцию аксессуаров NMOBI. Попробуйте выбрать
                  другую категорию или свяжитесь с нами в WhatsApp для подбора.
                </p>
              </div>
            ) : (
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-5">
                {filteredPublicProducts.map((product) => (
                  <div
                    key={product.id}
                    className="group rounded-3xl bg-white/95 border border-[#E7DDCD] overflow-hidden shadow-sm hover:shadow-xl transition-all flex flex-col justify-between"
                  >
                    <div>
                      <div className="relative h-56 bg-[#E7DDCD]/40 overflow-hidden">
                        <img
                          src={product.imageUrl}
                          alt={product.name}
                          className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500"
                        />
                        {categoryMap[product.categoryId] && (
                          <span className="absolute top-3 left-3 px-3 py-1 rounded-full text-[11px] font-medium bg-[#F4F0E8]/90 text-[#76543D] backdrop-blur-sm">
                            {categoryMap[product.categoryId]}
                          </span>
                        )}
                        {product.inStock === false && (
                          <span className="absolute top-3 right-3 px-3 py-1 rounded-full text-[11px] font-semibold bg-[#252525]/90 text-white">
                            Нет в наличии
                          </span>
                        )}
                      </div>

                      <div className="p-5 space-y-2">
                        <div className="flex items-start justify-between gap-2">
                          <h3 className="text-base font-bold text-[#252525] leading-snug">
                            {product.name}
                          </h3>
                          <span className="shrink-0 text-base font-bold text-[#76543D]">
                            {Number(product.price).toLocaleString('ru-RU')} сом
                          </span>
                        </div>
                        <p className="text-xs text-[#252525]/75 line-clamp-2 leading-relaxed">
                          {product.shortDescription}
                        </p>
                      </div>
                    </div>

                    <div className="px-5 pb-5 pt-2 flex items-center gap-2.5">
                      <button
                        onClick={() => setSelectedProductModal(product)}
                        className="flex-1 py-2.5 px-4 rounded-xl border border-[#E7DDCD] bg-[#F4F0E8]/70 text-xs font-medium text-[#252525] hover:bg-[#E7DDCD] transition-colors"
                      >
                        Подробнее
                      </button>
                      <button
                        disabled={product.inStock === false}
                        onClick={() => addToCart(product, true)}
                        className="flex-1 py-2.5 px-4 rounded-xl bg-[#252525] text-[#F4F0E8] text-xs font-medium hover:bg-[#C47B4A] transition-colors disabled:opacity-40 disabled:pointer-events-none"
                      >
                        Заказать
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {/* =============================================================== */}
        {/* TAB 4: ДОСТАВКА И НАШ АДРЕС                                     */}
        {/* =============================================================== */}
        {activeTab === 'delivery' && (
          <div className="space-y-8">
            <section className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-6 sm:p-8 shadow-sm space-y-6">
              <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-[#E7DDCD] text-[#76543D] text-xs font-medium">
                <Truck className="w-3.5 h-3.5" />
                <span>Условия получения</span>
              </div>

              <div className="space-y-2">
                <h1 className="text-2xl sm:text-3xl font-bold text-[#252525]">
                  Доставка
                </h1>
                <p className="text-sm sm:text-base text-[#76543D] font-medium">
                  «Мы заботимся о том, чтобы ваши покупки доходили быстро и
                  удобно».
                </p>
              </div>

              <div className="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
                {[
                  'Доставка через Яндекс Доставку',
                  'Возможность доставки по Бишкеку',
                  'Стоимость доставки рассчитывается отдельно',
                  'Уточнение адреса и времени доставки по телефону',
                ].map((point, idx) => (
                  <div
                    key={idx}
                    className="flex items-start gap-3 p-4 rounded-2xl bg-[#F4F0E8] border border-[#E7DDCD]"
                  >
                    <CheckCircle2 className="w-4 h-4 text-[#C47B4A] shrink-0 mt-0.5" />
                    <span className="text-xs sm:text-sm font-medium text-[#252525]">
                      {point}
                    </span>
                  </div>
                ))}
              </div>

              <div className="pt-2 flex flex-wrap gap-3">
                <button
                  onClick={() => {
                    if (cart.length > 0) {
                      setIsCartOpen(true);
                    } else {
                      setActiveTab('catalog');
                    }
                  }}
                  className="inline-flex items-center gap-2 px-5 py-3 rounded-2xl bg-[#252525] text-[#F4F0E8] text-xs sm:text-sm font-medium hover:bg-[#76543D] transition-colors"
                >
                  <ShoppingBag className="w-4 h-4 text-[#C47B4A]" />
                  <span>Заказать доставку</span>
                </button>

                <a
                  href="https://go.yandex/"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="inline-flex items-center gap-2 px-5 py-3 rounded-2xl bg-[#E7DDCD] text-[#252525] text-xs sm:text-sm font-medium hover:bg-[#C47B4A] hover:text-white transition-colors"
                >
                  <span>Сервис Яндекс Доставки (Яндекс Go)</span>
                  <ExternalLink className="w-3.5 h-3.5" />
                </a>
              </div>
            </section>

            {/* Наш адрес: г. Бишкек, проспект Чуй, дом 278 */}
            <section className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-6 sm:p-8 shadow-sm space-y-5">
              <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                <div className="space-y-1">
                  <div className="inline-flex items-center gap-1.5 text-xs font-semibold uppercase tracking-wider text-[#76543D]">
                    <MapPin className="w-3.5 h-3.5 text-[#C47B4A]" />
                    <span>Наш адрес</span>
                  </div>
                  <h2 className="text-xl sm:text-2xl font-bold text-[#252525]">
                    г. Бишкек, проспект Чуй, дом 278
                  </h2>
                </div>

                <div className="flex flex-wrap gap-2.5">
                  <a
                    href="https://yandex.ru/maps/?text=Бишкек+проспект+Чуй+278"
                    target="_blank"
                    rel="noopener noreferrer"
                    className="inline-flex items-center gap-1.5 px-4 py-2.5 rounded-xl bg-[#C47B4A] text-white text-xs font-medium hover:opacity-95 transition-opacity"
                  >
                    <MapPin className="w-3.5 h-3.5" />
                    <span>Построить маршрут</span>
                  </a>
                  <a
                    href="tel:+996704102050"
                    className="inline-flex items-center gap-1.5 px-4 py-2.5 rounded-xl bg-[#E7DDCD] text-[#252525] text-xs font-medium hover:bg-[#76543D] hover:text-white transition-colors"
                  >
                    <PhoneCall className="w-3.5 h-3.5" />
                    <span>Связаться</span>
                  </a>
                </div>
              </div>

              <div className="rounded-2xl overflow-hidden border border-[#E7DDCD] h-72 sm:h-80 bg-[#E7DDCD]/40">
                <iframe
                  title="Карта NMOBI — г. Бишкек, проспект Чуй, дом 278"
                  src="https://www.openstreetmap.org/export/embed.html?bbox=74.5650%2C42.8725%2C74.5820%2C42.8810&layer=mapnik&marker=42.8769%2C74.5736"
                  className="w-full h-full border-0"
                  loading="lazy"
                />
              </div>
            </section>
          </div>
        )}

        {/* =============================================================== */}
        {/* TAB 5: КОНТАКТЫ                                                 */}
        {/* =============================================================== */}
        {activeTab === 'contacts' && (
          <div className="space-y-8">
            <section className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-6 sm:p-8 shadow-sm space-y-6">
              <div className="space-y-2">
                <span className="text-xs font-semibold uppercase tracking-widest text-[#76543D]">
                  Связь с магазином
                </span>
                <h1 className="text-2xl sm:text-3xl font-bold text-[#252525]">
                  Контакты NMOBI
                </h1>
                <p className="text-sm text-[#252525]/80">
                  Ответим на ваши вопросы, поможем подобрать аксессуар и
                  согласуем удобное время доставки.
                </p>
              </div>

              <div className="p-5 rounded-2xl bg-[#F4F0E8] border border-[#E7DDCD] flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                <div>
                  <span className="text-xs text-[#76543D] block">
                    Основной номер телефона:
                  </span>
                  <a
                    href="tel:+996704102050"
                    className="text-2xl font-bold text-[#252525] tracking-wide hover:text-[#C47B4A] transition-colors"
                  >
                    0704 10 20 50
                  </a>
                  <span className="text-xs text-[#76543D]/80 block mt-0.5">
                    +996 704 10 20 50 • г. Бишкек, проспект Чуй, дом 278
                  </span>
                </div>

                <div className="flex flex-wrap gap-3">
                  <a
                    href="tel:+996704102050"
                    className="inline-flex items-center gap-2 px-5 py-3 rounded-2xl bg-[#252525] text-[#F4F0E8] text-xs font-medium hover:bg-[#76543D] transition-colors"
                  >
                    <PhoneCall className="w-4 h-4 text-[#C47B4A]" />
                    <span>Позвонить</span>
                  </a>

                  <a
                    href="https://wa.me/996704102050"
                    target="_blank"
                    rel="noopener noreferrer"
                    className="inline-flex items-center gap-2 px-5 py-3 rounded-2xl bg-[#C47B4A] text-white text-xs font-medium hover:opacity-95 transition-opacity"
                  >
                    <MessageCircle className="w-4 h-4" />
                    <span>Написать в WhatsApp</span>
                  </a>
                </div>
              </div>
            </section>

            {/* Вопросы покупателей */}
            <section className="rounded-3xl bg-white/85 border border-[#E7DDCD] p-6 sm:p-8 shadow-sm space-y-5">
              <h2 className="text-xl font-bold text-[#252525]">
                Вопросы покупателей
              </h2>

              <div className="space-y-3">
                {[
                  {
                    q: 'Как оформить заказ?',
                    a: 'Выберите понравившийся товар в разделе «Каталог», нажмите кнопку «Заказать», укажите ваше имя, номер телефона и адрес доставки по Бишкеку.',
                  },
                  {
                    q: 'Как осуществляется доставка?',
                    a: 'Мы отправляем заказы через сервис Яндекс Доставка по всему Бишкеку. Стоимость доставки рассчитывается отдельно по тарифу сервиса, а время уточняется по телефону.',
                  },
                  {
                    q: 'Где находится ваш магазин?',
                    a: 'Мы находимся по адресу: г. Бишкек, проспект Чуй, дом 278. Во вкладке «Доставка» вы можете открыть карту и построить маршрут.',
                  },
                  {
                    q: 'Можно ли проконсультироваться перед покупкой?',
                    a: 'Конечно! Позвоните по номеру 0704 10 20 50 или напишите нам в WhatsApp — мы поможем выбрать подходящий кабель, адаптер, чехол или защитное стекло.',
                  },
                ].map((faq, idx) => (
                  <div
                    key={idx}
                    className="p-4 rounded-2xl bg-[#F4F0E8]/90 border border-[#E7DDCD] space-y-1.5"
                  >
                    <h3 className="text-sm font-bold text-[#252525]">{faq.q}</h3>
                    <p className="text-xs sm:text-sm text-[#252525]/80 leading-relaxed">
                      {faq.a}
                    </p>
                  </div>
                ))}
              </div>
            </section>
          </div>
        )}
      </main>

      {/* =============================================================== */}
      {/* MODAL: ПОДРОБНЕЕ О ТОВАРЕ                                       */}
      {/* =============================================================== */}
      {selectedProductModal && (
        <div className="fixed inset-0 z-50 bg-[#252525]/60 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-[#F4F0E8] border border-[#E7DDCD] rounded-3xl max-w-lg w-full overflow-hidden shadow-2xl">
            <div className="relative h-64 bg-[#E7DDCD]">
              <img
                src={selectedProductModal.imageUrl}
                alt={selectedProductModal.name}
                className="w-full h-full object-cover"
              />
              <button
                onClick={() => setSelectedProductModal(null)}
                className="absolute top-4 right-4 w-9 h-9 rounded-full bg-[#252525]/80 text-white flex items-center justify-center hover:bg-[#252525]"
              >
                <X className="w-4 h-4" />
              </button>
            </div>

            <div className="p-6 space-y-4">
              <div className="flex items-start justify-between gap-3">
                <div>
                  {categoryMap[selectedProductModal.categoryId] && (
                    <span className="text-xs font-medium text-[#76543D]">
                      {categoryMap[selectedProductModal.categoryId]}
                    </span>
                  )}
                  <h3 className="text-xl font-bold text-[#252525]">
                    {selectedProductModal.name}
                  </h3>
                </div>
                <span className="text-lg font-bold text-[#C47B4A] shrink-0">
                  {Number(selectedProductModal.price).toLocaleString('ru-RU')} сом
                </span>
              </div>

              <p className="text-sm text-[#252525]/85 leading-relaxed">
                {selectedProductModal.fullDescription ||
                  selectedProductModal.shortDescription}
              </p>

              <div className="pt-2 flex items-center gap-3">
                <button
                  disabled={selectedProductModal.inStock === false}
                  onClick={() => addToCart(selectedProductModal, true)}
                  className="flex-1 py-3.5 rounded-2xl bg-[#252525] text-[#F4F0E8] text-sm font-medium hover:bg-[#C47B4A] transition-colors disabled:opacity-40"
                >
                  {selectedProductModal.inStock !== false
                    ? 'Заказать'
                    : 'Нет в наличии'}
                </button>
              </div>
            </div>
          </div>
        </div>
      )}

      {/* =============================================================== */}
      {/* MODAL: КОРЗИНА И ОФОРМЛЕНИЕ ЗАКАЗА                              */}
      {/* =============================================================== */}
      {isCartOpen && (
        <div className="fixed inset-0 z-50 bg-[#252525]/60 backdrop-blur-sm flex items-end sm:items-center justify-center p-0 sm:p-4">
          <div className="bg-[#F4F0E8] border-t sm:border border-[#E7DDCD] rounded-t-3xl sm:rounded-3xl max-w-lg w-full max-h-[90vh] overflow-y-auto p-6 space-y-5 shadow-2xl">
            <div className="flex items-center justify-between border-b border-[#E7DDCD] pb-4">
              <h3 className="text-lg font-bold text-[#252525]">Ваш заказ</h3>
              <button
                onClick={() => setIsCartOpen(false)}
                className="w-8 h-8 rounded-full bg-[#E7DDCD] flex items-center justify-center text-[#252525]"
              >
                <X className="w-4 h-4" />
              </button>
            </div>

            {orderSuccessData ? (
              <div className="py-8 text-center space-y-4">
                <div className="w-14 h-14 rounded-full bg-[#C47B4A]/15 text-[#C47B4A] mx-auto flex items-center justify-center">
                  <CheckCircle2 className="w-8 h-8" />
                </div>
                <h4 className="text-xl font-bold text-[#252525]">
                  Заказ успешно оформлен!
                </h4>
                <p className="text-xs sm:text-sm text-[#76543D] max-w-sm mx-auto leading-relaxed">
                  Сумма заказа:{' '}
                  <strong>
                    {Number(orderSuccessData.totalAmount).toLocaleString('ru-RU')}{' '}
                    сом
                  </strong>
                  . Мы свяжемся с вами по указанному номеру телефона для
                  подтверждения и расчёта доставки.
                </p>
                <button
                  onClick={() => {
                    setOrderSuccessData(null);
                    setIsCartOpen(false);
                  }}
                  className="px-6 py-3 rounded-2xl bg-[#252525] text-[#F4F0E8] text-xs font-medium"
                >
                  Вернуться в магазин
                </button>
              </div>
            ) : cart.length === 0 ? (
              <div className="py-10 text-center space-y-3">
                <p className="text-sm text-[#76543D]">Корзина пока пуста</p>
                <button
                  onClick={() => {
                    setIsCartOpen(false);
                    setActiveTab('catalog');
                  }}
                  className="px-5 py-2.5 rounded-xl bg-[#252525] text-[#F4F0E8] text-xs font-medium"
                >
                  Перейти в каталог
                </button>
              </div>
            ) : (
              <>
                <div className="space-y-3 max-h-56 overflow-y-auto pr-1">
                  {cart.map((item) => (
                    <div
                      key={item.productId}
                      className="flex items-center justify-between gap-3 p-3 rounded-2xl bg-white border border-[#E7DDCD]"
                    >
                      <div className="flex items-center gap-3">
                        <img
                          src={item.imageUrl}
                          alt={item.name}
                          className="w-12 h-12 rounded-xl object-cover bg-[#E7DDCD]"
                        />
                        <div>
                          <p className="text-xs font-bold text-[#252525]">
                            {item.name}
                          </p>
                          <p className="text-xs text-[#76543D]">
                            {Number(item.price).toLocaleString('ru-RU')} сом
                          </p>
                        </div>
                      </div>

                      <div className="flex items-center gap-2">
                        <button
                          type="button"
                          onClick={() => updateCartQuantity(item.productId, -1)}
                          className="w-7 h-7 rounded-lg bg-[#E7DDCD] flex items-center justify-center"
                        >
                          <Minus className="w-3 h-3" />
                        </button>
                        <span className="text-xs font-bold w-5 text-center">
                          {item.quantity}
                        </span>
                        <button
                          type="button"
                          onClick={() => updateCartQuantity(item.productId, 1)}
                          className="w-7 h-7 rounded-lg bg-[#E7DDCD] flex items-center justify-center"
                        >
                          <Plus className="w-3 h-3" />
                        </button>
                      </div>
                    </div>
                  ))}
                </div>

                <div className="flex items-center justify-between pt-2 border-t border-[#E7DDCD]">
                  <span className="text-sm font-medium text-[#76543D]">
                    Общая сумма:
                  </span>
                  <span className="text-lg font-bold text-[#252525]">
                    {cartTotal.toLocaleString('ru-RU')} сом
                  </span>
                </div>

                <form onSubmit={handleCreateOrder} className="space-y-3 pt-2">
                  {orderFormError && (
                    <div className="p-3 rounded-xl bg-red-100 text-red-800 text-xs font-medium">
                      {orderFormError}
                    </div>
                  )}

                  <input
                    type="text"
                    required
                    value={customerName}
                    onChange={(e) => setCustomerName(e.target.value)}
                    placeholder="Ваше имя *"
                    className="w-full px-4 py-3 rounded-xl bg-white border border-[#E7DDCD] text-xs sm:text-sm focus:outline-none focus:border-[#76543D]"
                  />
                  <input
                    type="tel"
                    required
                    value={customerPhone}
                    onChange={(e) => setCustomerPhone(e.target.value)}
                    placeholder="Номер телефона (например: 0704 10 20 50) *"
                    className="w-full px-4 py-3 rounded-xl bg-white border border-[#E7DDCD] text-xs sm:text-sm focus:outline-none focus:border-[#76543D]"
                  />
                  <input
                    type="text"
                    required
                    value={deliveryAddress}
                    onChange={(e) => setDeliveryAddress(e.target.value)}
                    placeholder="Адрес доставки по Бишкеку (или Самовывоз) *"
                    className="w-full px-4 py-3 rounded-xl bg-white border border-[#E7DDCD] text-xs sm:text-sm focus:outline-none focus:border-[#76543D]"
                  />
                  <textarea
                    rows={2}
                    value={orderComment}
                    onChange={(e) => setOrderComment(e.target.value)}
                    placeholder="Комментарий к заказу (необязательно)"
                    className="w-full px-4 py-2.5 rounded-xl bg-white border border-[#E7DDCD] text-xs sm:text-sm focus:outline-none focus:border-[#76543D]"
                  />

                  <button
                    type="submit"
                    disabled={submittingOrder}
                    className="w-full py-3.5 rounded-2xl bg-[#C47B4A] text-white font-semibold text-sm hover:opacity-95 transition-opacity disabled:opacity-50"
                  >
                    {submittingOrder ? 'Отправка заказа...' : 'Оформить заказ'}
                  </button>
                </form>
              </>
            )}
          </div>
        </div>
      )}

      {/* =============================================================== */}
      {/* 10. НИЖНЕЕ МЕНЮ САЙТА (БЕЗ АДМИН-ПАНЕЛИ)                        */}
      {/* =============================================================== */}
      <nav className="fixed bottom-4 inset-x-0 z-40 px-4 pointer-events-none">
        <div className="max-w-md mx-auto bg-[#252525]/95 backdrop-blur-md text-[#F4F0E8] rounded-2xl p-1.5 shadow-2xl border border-white/10 flex items-center justify-between pointer-events-auto">
          {[
            { id: 'home', label: 'Главная', icon: Home },
            { id: 'about', label: 'О магазине', icon: Info },
            { id: 'catalog', label: 'Каталог', icon: SlidersHorizontal },
            { id: 'delivery', label: 'Доставка', icon: Truck },
            { id: 'contacts', label: 'Контакты', icon: Phone },
          ].map((item) => {
            const Icon = item.icon;
            const active = activeTab === item.id;
            return (
              <button
                key={item.id}
                onClick={() => setActiveTab(item.id)}
                className={`flex-1 py-2 px-1 rounded-xl flex flex-col items-center gap-1 transition-all ${
                  active
                    ? 'bg-[#C47B4A] text-white shadow-sm'
                    : 'text-[#E7DDCD]/75 hover:text-white'
                }`}
              >
                <Icon className="w-4 h-4" />
                <span className="text-[10px] font-medium">{item.label}</span>
              </button>
            );
          })}
        </div>
      </nav>
    </div>
  );
}
