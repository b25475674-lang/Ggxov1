# Ggxov1
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
import { 
  getAuth, 
  createUserWithEmailAndPassword, 
  signInWithEmailAndPassword, 
  signOut, 
  onAuthStateChanged 
} from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";
import { 
  getFirestore, 
  collection, 
  addDoc, 
  getDocs, 
  serverTimestamp,
  query,
  orderBy 
} from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

// ⚠️ วาง Firebase Config ของคุณตรงนี้
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

let currentMode = 'login'; // 'login' หรือ 'register'

// เปิด/ปิด Modal
window.openModal = (mode) => {
  currentMode = mode;
  document.getElementById('modal-title').innerText = mode === 'login' ? 'เข้าสู่ระบบ' : 'สมัครสมาชิก';
  document.getElementById('auth-submit-btn').innerText = mode === 'login' ? 'เข้าสู่ระบบ' : 'ลงทะเบียน';
  document.getElementById('auth-modal').classList.remove('hidden');
  document.getElementById('auth-modal').classList.add('flex');
};

window.closeModal = () => {
  document.getElementById('auth-modal').classList.add('hidden');
  document.getElementById('auth-modal').classList.remove('flex');
};

// จัดการสมัครสมาชิก / เข้าสู่ระบบ
document.getElementById('auth-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const email = document.getElementById('auth-email').value;
  const password = document.getElementById('auth-password').value;

  try {
    if (currentMode === 'register') {
      await createUserWithEmailAndPassword(auth, email, password);
      alert('สมัครสมาชิกสำเร็จ!');
    } else {
      await signInWithEmailAndPassword(auth, email, password);
      alert('เข้าสู่ระบบสำเร็จ!');
    }
    closeModal();
  } catch (error) {
    alert('เกิดข้อผิดพลาด: ' + error.message);
  }
});

// ออกจากระบบ
window.logout = () => signOut(auth);

// ตรวจสอบสถานะการเข้าสู่ระบบ
onAuthStateChanged(auth, (user) => {
  const authSection = document.getElementById('auth-section');
  const userSection = document.getElementById('user-section');
  const addModCard = document.getElementById('add-mod-card');
  const userEmail = document.getElementById('user-email');

  if (user) {
    authSection.classList.add('hidden');
    userSection.classList.remove('hidden');
    addModCard.classList.remove('hidden');
    userEmail.innerText = user.email;
  } else {
    authSection.classList.remove('hidden');
    userSection.classList.add('hidden');
    addModCard.classList.add('hidden');
  }
});

// จัดการการเพิ่ม Mod
document.getElementById('add-mod-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const title = document.getElementById('mod-title').value;
  const desc = document.getElementById('mod-desc').value;
  const link = document.getElementById('mod-link').value;

  try {
    await addDoc(collection(db, "mods"), {
      title,
      desc,
      link,
      author: auth.currentUser.email,
      createdAt: serverTimestamp()
    });
    alert('เพิ่ม Mod เรียบร้อยแล้ว!');
    document.getElementById('add-mod-form').reset();
    loadMods();
  } catch (error) {
    alert('ไม่สามารถเพิ่ม Mod ได้: ' + error.message);
  }
});

// ดึงรายการ Mod มาแสดง
async function loadMods() {
  const modList = document.getElementById('mod-list');
  modList.innerHTML = '<p class="text-slate-500 col-span-full">กำลังโหลดข้อมูล...</p>';

  try {
    const q = query(collection(db, "mods"), orderBy("createdAt", "desc"));
    const querySnapshot = await getDocs(q);
    
    modList.innerHTML = '';
    if (querySnapshot.empty) {
      modList.innerHTML = '<p class="text-slate-500 col-span-full">ยังไม่มี Mod ในระบบ</p>';
      return;
    }

    querySnapshot.forEach((doc) => {
      const data = doc.data();
      const card = document.createElement('div');
      card.className = 'bg-slate-800 border border-slate-700 p-5 rounded-xl flex flex-col justify-between hover:border-slate-600 transition';
      card.innerHTML = `
        <div>
          <h3 class="text-lg font-bold text-white mb-1">${data.title}</h3>
          <p class="text-slate-400 text-sm mb-4 whitespace-pre-line">${data.desc}</p>
        </div>
        <div>
          <p class="text-xs text-slate-500 mb-3">แชร์โดย: ${data.author}</p>
          <a href="${data.link}" target="_blank" rel="noopener noreferrer" class="block text-center bg-green-600 hover:bg-green-500 text-white font-medium py-2 px-4 rounded-lg transition">
            ⬇️ ดาวน์โหลด Mod
          </a>
        </div>
      `;
      modList.appendChild(card);
    });
  } catch (error) {
    console.error("Error loading mods:", error);
    modList.innerHTML = '<p class="text-red-400 col-span-full">เกิดข้อผิดพลาดในการโหลดข้อมูล</p>';
  }
}

// โหลดข้อมูลเมื่อเปิดหน้าเว็บ
loadMods();
