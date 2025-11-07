<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    
    <!-- === AAPKI DI HUI META LINE YAHAN ADD KAR DI GAYI HAI === -->
    <meta name='viewport' content='width=device-width, initial-scale=1.0, user-scalable=no, shrink-to-fit=no'/>
    
    <title>View2Earn</title>
    
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    
    <!-- === AAPKA MONETAG AD CODE === -->
    <script src='//libtl.com/sdk.js' data-zone='10150753' data-sdk='show_10150753'></script>

    <!-- Firebase SDKs -->
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-storage.js"></script>

    <!-- CSS and Fonts -->
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
    
    <style>
        :root { --primary: #4299E1; --background: #F7FAFC; --card-bg: #FFFFFF; --text-main: #2D3748; --text-secondary: #718096; }
        body { background-color: var(--background); color: var(--text-main); font-family: 'Poppins', sans-serif; }
        #particles-js { position: fixed; width: 100%; height: 100%; top: 0; left: 0; z-index: 0; }
        .card { background: var(--card-bg); border-radius: 16px; box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1); position: relative; z-index: 1; }
        .btn-primary { background-color: var(--primary); color: white; box-shadow: 0 4px 14px 0 rgba(66, 153, 225, 0.39); transition: all 0.3s ease; border-radius: 9999px; font-weight: bold; padding: 0.75rem 2rem; }
        .btn-primary:hover { transform: translateY(-1px); box-shadow: 0 6px 20px 0 rgba(66, 153, 225, 0.5); }
        .btn-primary:disabled { background-color: #A0AEC0; cursor: not-allowed; box-shadow: none; }
        .fade-in { animation: fadeIn 0.5s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .nav-item.active { color: var(--primary); }
        .glass-nav { background: rgba(255, 255, 255, 0.8); backdrop-filter: blur(10px); border-top: 1px solid #E2E8F0; }
    </style>
</head>
<body class="pb-24">
    <div id="particles-js"></div>
    <div id="app" class="container mx-auto px-4 py-6 relative z-10">
        <h1 class="text-center text-2xl mt-48 text-gray-500">App Start ho raha hai...</h1>
    </div>
    <div id="bottom-nav" class="fixed bottom-0 left-0 right-0 glass-nav flex justify-around py-3 text-sm z-50" style="display: none;">
        <button onclick="navigate('home')" id="nav-home" class="nav-item flex flex-col items-center active"><i class="fas fa-home text-lg mb-1"></i><span>Home</span></button>
        <button onclick="navigate('earn')" id="nav-earn" class="nav-item flex flex-col items-center"><i class="fas fa-coins text-lg mb-1"></i><span>Earn</span></button>
        <button onclick="navigate('withdraw')" id="nav-withdraw" class="nav-item flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>Withdraw</span></button>
        <button onclick="navigate('profile')" id="nav-profile" class="nav-item flex flex-col items-center"><i class="fas fa-user text-lg mb-1"></i><span>Profile</span></button>
    </div>

<script src="https://cdn.jsdelivr.net/particles.js/2.0.0/particles.min.js"></script>

<script>
// --- AAPKI FIREBASE DETAILS YAHAN HAIN ---
const firebaseConfig = {
  apiKey: "AIzaSyBakB9u12hokftJWmWVjkqXO1vWuf94dBs",
  authDomain: "view2earn-c317e.firebaseapp.com",
  projectId: "view2earn-c317e",
  storageBucket: "view2earn-c317e.appspot.com",
  messagingSenderId: "938039197586",
  appId: "1:938039197586:web:4273dc705eae7bb6e44b4b"
};
// --- FIREBASE DETAILS END ---

document.addEventListener('DOMContentLoaded', () => {
    
    const appDiv = document.getElementById('app');
    const bottomNav = document.getElementById('bottom-nav');
    let db, storage;
    let userData = {};
    let appSettings = { taskReward: 1, minWithdraw: 100 };
    const tg = window.Telegram.WebApp;
    const user = tg.initDataUnsafe?.user || { id: 'guest', first_name: 'Guest' };
    let currentPage = 'home';

    async function startApp() {
        try {
            firebase.initializeApp(firebaseConfig);
            db = firebase.firestore();
            storage = firebase.storage();
            tg.expand();
            
            const settingsDoc = await db.collection('settings').doc('appConfig').get();
            if (settingsDoc.exists) { appSettings = { ...appSettings, ...settingsDoc.data() }; }

            if (user.id === 'guest') { renderAuth(false); return; }

            const userDoc = await db.collection('users').doc(String(user.id)).get();
            renderAuth(userDoc.exists);
        } catch (error) {
            console.error("CRITICAL ERROR:", error);
            appDiv.innerHTML = `<div class="text-center mt-40"><h1 class="text-2xl font-bold text-red-500">Oops! Connection Error</h1><p class="text-gray-600 mt-2">Server se connect nahi ho pa raha hai. Please check your Firebase details and rules.</p><p class="text-xs text-gray-400 mt-4">${error.message}</p></div>`;
        }
    }

    function renderAuth(userExists) {
        const title = userExists ? "Welcome Back!" : "Create Your Account";
        const subtitle = userExists ? user.first_name : "Welcome to View2Earn!";
        const buttonText = userExists ? "Login" : "Create Account";
        const action = userExists ? "handleLogin(this)" : "handleSignup(this)";
        appDiv.innerHTML = `<div class="fade-in text-center" style="margin-top: 30vh;"><h1 class="text-4xl font-bold text-gray-800">${title}</h1><p class="text-xl text-gray-500 mt-2">${subtitle}</p><button onclick="${action}" class="mt-8 btn-primary text-lg">${buttonText}</button></div>`;
    }

    window.handleLogin = async function(button) {
        button.disabled = true; button.innerHTML = 'Loading...';
        const doc = await db.collection('users').doc(String(user.id)).get();
        userData = doc.data();
        renderApp();
    }

    window.handleSignup = async function(button) {
        button.disabled = true; button.innerHTML = 'Creating Account...';
        const newUserData = {
            balance: 0, totalAds: 0, photoURL: `https://i.pravatar.cc/150?u=${user.id}`,
            username: user.username || 'N/A', firstName: user.first_name || 'User', joinedAt: new Date()
        };
        await db.collection('users').doc(String(user.id)).set(newUserData);
        userData = newUserData;
        renderApp();
    }
    
    function renderApp() {
        bottomNav.style.display = 'flex';
        navigate('home');
    }

    window.navigate = function(page) {
        currentPage = page;
        document.querySelectorAll('.nav-item').forEach(btn => btn.classList.remove('active'));
        document.getElementById('nav-' + page).classList.add('active');
        let content = '';
        switch (page) {
            case 'home': content = getHomeHTML(); break;
            case 'earn': content = getEarnHTML(); break;
            case 'withdraw': content = getWithdrawHTML(); break;
            case 'profile': content = getProfileHTML(); break;
        }
        appDiv.innerHTML = `<div class="fade-in">${content}</div>`;
    }

    function getHomeHTML() {
        return `<div class="card p-6 text-center"><p class="text-lg mb-1 text-gray-600">Current Balance</p><p class="text-5xl font-bold text-gray-800">₹${userData.balance?.toFixed(2) || '0.00'}</p></div><div class="card p-6 mt-6"><h3 class="text-xl font-bold mb-2">Start Earning!</h3><p class="text-gray-600 mb-4">Task center mein jaakar naye tasks poore karein.</p><button onclick="navigate('earn')" class="w-full font-bold py-3 px-4 rounded-full btn-primary"><i class="fas fa-coins mr-2"></i>Go to Task Center</button></div><p class="text-center text-sm text-gray-400 mt-8">View2Earn Bot made by Aditya</p>`;
    }

    function getEarnHTML() {
        return `<div class="card p-6"><h2 class="text-2xl font-bold mb-4">Task Center</h2><div class="p-4 rounded-xl border border-gray-200"><div class="flex justify-between items-center mb-3"><h3 class="font-semibold text-lg">Watch an Ad</h3><span class="font-bold text-green-500 text-lg">+₹${appSettings.taskReward.toFixed(2)}</span></div><p class="text-gray-600 mb-4 text-sm bg-blue-50 p-3 rounded-lg"><i class="fas fa-info-circle mr-2 text-blue-500"></i>Button par click karne ke baad ek ad dikhega. Reward paane ke liye ad ko poora dekhein ya 'Close' button ke aane par use band karein.</p><button onclick="watchAd(this)" class="w-full font-bold py-3 px-4 rounded-lg btn-primary"><i class="fas fa-play mr-2"></i>Watch Ad Now</button></div></div>`;
    }
    
    function getWithdrawHTML() {
        return `<div class="card p-6"><h2 class="text-2xl font-bold mb-4">Withdraw Funds</h2><div class="text-center mb-6 p-4 bg-gray-50 rounded-lg"><p class="text-gray-600">Available Balance</p><p class="text-4xl font-bold text-green-500">₹${userData.balance?.toFixed(2) || '0.00'}</p><p class="text-sm text-gray-500 mt-1">Minimum withdrawal: ₹${appSettings.minWithdraw || 100}</p></div><div class="space-y-4"><div><label class="block text-sm font-medium mb-2 text-gray-700">Payment Method</label><select id="method" class="w-full p-3 bg-gray-100 border border-gray-300 rounded-lg"><option value="UPI">UPI</option><option value="Paytm Wallet">Paytm Wallet</option></select></div><div><label class="block text-sm font-medium mb-2 text-gray-700">Amount (₹)</label><input type="number" id="amount" class="w-full p-3 bg-gray-100 border border-gray-300 rounded-lg" placeholder="e.g., 100"></div><div><label class="block text-sm font-medium mb-2 text-gray-700">Aapki UPI ID / Phone Number</label><input type="text" id="wallet_id" class="w-full p-3 bg-gray-100 border border-gray-300 rounded-lg" placeholder="Enter details here"></div><button onclick="handleWithdraw(this)" class="w-full text-white font-bold py-3 px-4 rounded-full btn-primary"><i class="fas fa-paper-plane mr-2"></i>Submit Request</button></div></div>`;
    }

    function getProfileHTML() {
        return `<div class="card p-6 text-center"><img src="${userData.photoURL || 'https://i.pravatar.cc/150'}" class="w-24 h-24 rounded-full border-4 border-blue-400 mx-auto object-cover"/><h2 class="text-xl font-bold mt-3">${userData.firstName || 'User'}</h2><p class="text-gray-500">@${userData.username || 'guest'}</p><div class="mt-4"><label for="photoUpload" class="cursor-pointer bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-2 px-4 rounded-full text-sm">Change Picture</label><input id="photoUpload" type="file" class="hidden" accept="image/*" onchange="handlePhotoUpload(this)"></div></div><div class="card p-6 mt-6"><h3 class="text-lg font-bold mb-4">Your Stats</h3><div class="space-y-3"><div class="flex justify-between items-center text-gray-700"><span class="text-gray-500">Total Ads Watched</span><span class="font-bold">${userData.totalAds || 0}</span></div><div class="flex justify-between items-center text-gray-700"><span class="text-gray-500">Total Balance</span><span class="font-bold text-green-500">₹${userData.balance?.toFixed(2) || '0.00'}</span></div></div></div>`;
    }

    window.watchAd = function(button) {
        button.disabled = true; button.innerHTML = '<i class="fas fa-spinner fa-spin mr-2"></i>Loading Ad...';
        show_10150753().then(() => {
            rewardUser();
        }).catch(e => {
            console.error('Ad Error:', e);
            alert('Ad nahi chal paya. Please try again.');
            button.disabled = false; button.innerHTML = '<i class="fas fa-play mr-2"></i>Watch Ad Now';
        });
    }

    window.rewardUser = async function() {
        userData.balance = (userData.balance || 0) + appSettings.taskReward;
        userData.totalAds = (userData.totalAds || 0) + 1;
        await db.collection('users').doc(String(user.id)).update({ balance: userData.balance, totalAds: userData.totalAds });
        alert(`Badhai ho! Aapne ₹${appSettings.taskReward.toFixed(2)} kamaye hain.`);
        navigate(currentPage);
    }
    
    window.handleWithdraw = async function(button) {
        button.disabled = true; button.innerHTML = 'Submitting...';
        const amount = parseFloat(document.getElementById('amount').value);
        const walletId = document.getElementById('wallet_id').value;
        const minWithdraw = appSettings.minWithdraw || 100;
        if (!amount || !walletId) { alert('Please fill all fields.'); button.disabled = false; button.innerHTML = 'Submit Request'; return; }
        if (amount < minWithdraw) { alert(`Minimum withdrawal is ₹${minWithdraw}.`); button.disabled = false; button.innerHTML = 'Submit Request'; return; }
        if (amount > userData.balance) { alert('Aapke paas itna balance nahi hai.'); button.disabled = false; button.innerHTML = 'Submit Request'; return; }
        try {
            await db.collection('withdrawals').add({ userId: user.id, username: user.username, amount, walletId, method: document.getElementById('method').value, status: 'pending', timestamp: new Date() });
            userData.balance -= amount;
            await db.collection('users').doc(String(user.id)).update({ balance: userData.balance });
            alert('Withdrawal request submitted successfully!');
            navigate('home');
        } catch (e) { alert('Error submitting request. Please try again.'); button.disabled = false; button.innerHTML = 'Submit Request'; }
    }

    window.handlePhotoUpload = async function(input) {
        const file = input.files[0]; if (!file) return;
        const uploadLabel = input.previousElementSibling; uploadLabel.innerText = 'Uploading...';
        try {
            const fileRef = storage.ref(`profile_pictures/${user.id}`);
            const uploadTask = await fileRef.put(file);
            const downloadURL = await uploadTask.ref.getDownloadURL();
            await db.collection('users').doc(String(user.id)).update({ photoURL: downloadURL });
            userData.photoURL = downloadURL;
            alert('Profile picture updated!');
            navigate('profile');
        } catch(e) { alert('Error uploading photo.'); uploadLabel.innerText = 'Change Picture'; }
    }

    startApp();
    particlesJS("particles-js", {
        "particles": { "number": {"value": 80}, "color": {"value": "#4299E1"}, "shape": {"type": "circle"}, "opacity": {"value": 0.5}, "size": {"value": 3}, "line_linked": {"enable": true, "distance": 150, "color": "#4299E1", "opacity": 0.2}, "move": {"enable": true, "speed": 2}},
        "interactivity": {"events": {"onhover": {"enable": true, "mode": "grab"}}, "modes": {"grab": {"distance": 140}}}
    });
});
</script>
</body>
</html>
