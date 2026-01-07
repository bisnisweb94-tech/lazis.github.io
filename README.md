import React, { useState, useEffect } from 'react';
import { PlusCircle, Users, TrendingUp, TrendingDown, Wallet, DollarSign, BarChart3, Settings, FileText, Copy, X, LogOut, User, Lock, Eye, EyeOff, MapPin, Target, Edit2, Download } from 'lucide-react';

// Utility functions
const loadData = async (key, defaultValue) => {
  try {
    const result = await window.storage.get(key);
    return result ? JSON.parse(result.value) : defaultValue;
  } catch (error) {
    return defaultValue;
  }
};

const saveData = async (key, value) => {
  try {
    await window.storage.set(key, JSON.stringify(value));
  } catch (error) {
    console.error('Save error:', error);
  }
};

export default function MasjidZakatApp() {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [currentUser, setCurrentUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [showLogin, setShowLogin] = useState(false);
  
  useEffect(() => {
    const checkAuth = async () => {
      try {
        const session = await loadData('masjid-session', null);
        if (session && session.user) {
          setIsAuthenticated(true);
          setCurrentUser(session.user);
        }
      } catch (error) {
        console.log('Session check error:', error);
      }
      setLoading(false);
    };
    checkAuth();
  }, []);

  const handleLogin = async (user) => {
    try {
      setIsAuthenticated(true);
      setCurrentUser(user);
      setShowLogin(false);
      await saveData('masjid-session', { user, timestamp: Date.now() });
    } catch (error) {
      console.error('Login error:', error);
      alert('Terjadi kesalahan saat login. Silakan coba lagi.');
    }
  };

  const handleLogout = async () => {
    setIsAuthenticated(false);
    setCurrentUser(null);
    await saveData('masjid-session', null);
  };

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-emerald-50 to-teal-50">
        <div className="text-2xl text-emerald-700 font-medium">Memuat...</div>
      </div>
    );
  }

  return (
    <>
      <MainApp 
        currentUser={currentUser} 
        onLogout={handleLogout}
        onShowLogin={() => setShowLogin(true)}
        isAuthenticated={isAuthenticated}
      />
      {showLogin && <LoginPage onLogin={handleLogin} onClose={() => setShowLogin(false)} />}
    </>
  );
}

function LoginPage({ onLogin, onClose }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [showPassword, setShowPassword] = useState(false);
  const [error, setError] = useState('');
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const loadUsers = async () => {
      const storedUsers = await loadData('masjid-users', []);
      if (storedUsers.length === 0) {
        const defaultUsers = [
          { id: 1, username: 'admin', password: 'admin123', nama: 'Administrator', role: 'Admin' },
          { id: 2, username: 'petugas1', password: 'petugas123', nama: 'Petugas 1', role: 'Petugas' }
        ];
        await saveData('masjid-users', defaultUsers);
        setUsers(defaultUsers);
      } else {
        setUsers(storedUsers);
      }
    };
    loadUsers();
  }, []);

  const handleSubmit = async () => {
    try {
      setError('');
      const user = users.find(u => u.username === username && u.password === password);
      if (user) {
        await onLogin({ id: user.id, username: user.username, nama: user.nama, role: user.role });
      } else {
        setError('Username atau password salah!');
      }
    } catch (error) {
      console.error('Submit error:', error);
      setError('Terjadi kesalahan. Silakan coba lagi.');
    }
  };

  return (
    <div className="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
      <div className="bg-white rounded-3xl shadow-2xl max-w-md w-full p-8 relative">
        <button
          onClick={onClose}
          className="absolute top-4 right-4 text-gray-400 hover:text-gray-600"
        >
          <X className="w-6 h-6" />
        </button>

        <div className="text-center mb-8">
          <div className="inline-flex items-center justify-center w-20 h-20 bg-gradient-to-br from-emerald-600 to-teal-600 rounded-2xl mb-4">
            <Wallet className="w-10 h-10 text-white" />
          </div>
          <h1 className="text-3xl font-bold text-gray-800">Login Petugas</h1>
          <p className="text-gray-600">Masjid Baitul Hikmah</p>
        </div>

        <div className="space-y-4">
          <div>
            <label className="block text-sm font-semibold text-gray-700 mb-2">Username</label>
            <input
              type="text"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              onKeyPress={(e) => e.key === 'Enter' && handleSubmit()}
              className="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500"
              placeholder="Masukkan username"
            />
          </div>

          <div>
            <label className="block text-sm font-semibold text-gray-700 mb-2">Password</label>
            <div className="relative">
              <input
                type={showPassword ? "text" : "password"}
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                onKeyPress={(e) => e.key === 'Enter' && handleSubmit()}
                className="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500"
                placeholder="Masukkan password"
              />
              <button
                type="button"
                onClick={() => setShowPassword(!showPassword)}
                className="absolute right-3 top-3 text-gray-400"
              >
                {showPassword ? <EyeOff className="w-5 h-5" /> : <Eye className="w-5 h-5" />}
              </button>
            </div>
          </div>

          {error && (
            <div className="bg-red-50 border-2 border-red-200 text-red-700 px-4 py-3 rounded-xl text-sm">
              {error}
            </div>
          )}

          <button
            onClick={handleSubmit}
            className="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-3 rounded-xl hover:from-emerald-700 hover:to-teal-700 font-semibold shadow-lg"
          >
            Login
          </button>
        </div>

        <div className="mt-6 text-xs text-gray-500 text-center">
          <p>Admin: admin / admin123</p>
          <p>Petugas: petugas1 / petugas123</p>
        </div>
      </div>
    </div>
  );
}

function MainApp({ currentUser, onLogout, onShowLogin, isAuthenticated }) {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [data, setData] = useState({
    penerimaan: [],
    pengeluaran: [],
    mustahik: [],
    settings: {
      namaMasjid: 'Masjid Jami Baitul Hikmah',
      targetZakatFitrah: 0,
      tanggalDistribusi: '',
      jadwalMasjid: '',
      jadwalSanur: '',
      jadwalTampakSiring: '',
      jadwalNusaDua: '',
      namaBank: 'Bank Syariah Indonesia',
      kodeBank: '451',
      noRekening: '7019291698',
      atasNama: 'Eko Andri QQ BAITUL HIKMAH',
      noWAKonsultasi: '6289601705274',
      kategoriPengeluaran: ['Distribusi Zakat', 'Program Infak', 'Operasional Masjid', 'Lainnya']
    }
  });
  const [showModal, setShowModal] = useState({ type: null, data: null });

  useEffect(() => {
    const loadAllData = async () => {
      const penerimaan = await loadData('masjid-penerimaan', []);
      const pengeluaran = await loadData('masjid-pengeluaran', []);
      const mustahik = await loadData('masjid-mustahik', []);
      const settings = await loadData('masjid-settings', {
        namaMasjid: 'Masjid Jami Baitul Hikmah',
        targetZakatFitrah: 0,
        tanggalDistribusi: '',
        jadwalMasjid: '',
        jadwalSanur: '',
        jadwalTampakSiring: '',
        jadwalNusaDua: '',
        namaBank: 'Bank Syariah Indonesia',
        kodeBank: '451',
        noRekening: '7019291698',
        atasNama: 'Eko Andri QQ BAITUL HIKMAH',
        noWAKonsultasi: '6289601705274',
        kategoriPengeluaran: ['Distribusi Zakat', 'Program Infak', 'Operasional Masjid', 'Lainnya']
      });
      const kroscekCash = await loadData('masjid-kroscek-cash', null);
      const kroscekBank = await loadData('masjid-kroscek-bank', 0);
      
      setData({ 
        penerimaan, 
        pengeluaran, 
        mustahik, 
        settings,
        kroscekCash: kroscekCash || {
          100000: 0, 50000: 0, 20000: 0, 10000: 0, 5000: 0,
          2000: 0, 1000: 0, 500: 0, 200: 0, 100: 0
        },
        kroscekBank: kroscekBank || 0
      });
    };
    loadAllData();
  }, []);

  const handleSave = async (type, newData) => {
    if (type === 'penerimaan') {
      const updated = showModal.data 
        ? data.penerimaan.map(p => p.id === showModal.data.id ? { ...newData, id: showModal.data.id } : p)
        : [{ ...newData, id: Date.now(), petugas: currentUser?.nama || 'System' }, ...data.penerimaan];
      setData({ ...data, penerimaan: updated });
      await saveData('masjid-penerimaan', updated);
    } else if (type === 'pengeluaran') {
      const updated = showModal.data 
        ? data.pengeluaran.map(p => p.id === showModal.data.id ? { ...newData, id: showModal.data.id } : p)
        : [{ ...newData, id: Date.now(), petugas: currentUser?.nama || 'System' }, ...data.pengeluaran];
      setData({ ...data, pengeluaran: updated });
      await saveData('masjid-pengeluaran', updated);
    } else if (type === 'mustahik') {
      const updated = showModal.data 
        ? data.mustahik.map(m => m.id === showModal.data.id ? { ...newData, id: showModal.data.id } : m)
        : [{ ...newData, id: Date.now(), status: 'Aktif' }, ...data.mustahik];
      setData({ ...data, mustahik: updated });
      await saveData('masjid-mustahik', updated);
    } else if (type === 'settings') {
      setData({ ...data, settings: newData });
      await saveData('masjid-settings', newData);
    } else if (type === 'kroscekCash') {
      setData({ ...data, kroscekCash: newData });
      await saveData('masjid-kroscek-cash', newData);
    } else if (type === 'kroscekBank') {
      setData({ ...data, kroscekBank: newData });
      await saveData('masjid-kroscek-bank', newData);
    }
    setShowModal({ type: null, data: null });
  };

  const handleDelete = async (type, id) => {
    if (confirm('Hapus data ini?')) {
      if (type === 'penerimaan') {
        const updated = data.penerimaan.filter(p => p.id !== id);
        setData({ ...data, penerimaan: updated });
        await saveData('masjid-penerimaan', updated);
      } else if (type === 'pengeluaran') {
        const updated = data.pengeluaran.filter(p => p.id !== id);
        setData({ ...data, pengeluaran: updated });
        await saveData('masjid-pengeluaran', updated);
      } else if (type === 'mustahik') {
        const updated = data.mustahik.filter(m => m.id !== id);
        setData({ ...data, mustahik: updated });
        await saveData('masjid-mustahik', updated);
      }
    }
  };

  const formatRupiah = (amount) => {
    return new Intl.NumberFormat('id-ID', {
      style: 'currency',
      currency: 'IDR',
      minimumFractionDigits: 0
    }).format(amount);
  };

  const totalPenerimaan = data.penerimaan.reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const totalPengeluaran = data.pengeluaran.reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const saldo = totalPenerimaan - totalPengeluaran;
  const totalMuzaki = data.penerimaan.length;

  const totalZakatFitrah = data.penerimaan.filter(p => p.jenis === 'Zakat Fitrah').reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const totalZakatMaal = data.penerimaan.filter(p => p.jenis === 'Zakat Maal').reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const totalZakatProfesi = data.penerimaan.filter(p => p.jenis === 'Zakat Profesi').reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const totalInfak = data.penerimaan.filter(p => p.jenis === 'Infak').reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const totalFidyah = data.penerimaan.filter(p => p.jenis === 'Fidyah').reduce((sum, p) => sum + (p.jumlah || 0), 0);

  const berasFidyah = data.penerimaan.filter(p => p.jenisBeras === 'Fidyah').reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);
  const berasFitrah = data.penerimaan.filter(p => p.jenisBeras === 'Fitrah').reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);
  const berasSedekah = data.penerimaan.filter(p => p.jenisBeras === 'Sedekah').reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);
  const totalBeras = berasFidyah + berasFitrah + berasSedekah;

  return (
    <div className="min-h-screen bg-gradient-to-br from-emerald-50 to-cyan-50">
      <header className="bg-gradient-to-r from-emerald-600 to-teal-600 text-white shadow-xl">
        <div className="max-w-7xl mx-auto px-4 py-6">
          <div className="flex items-center justify-between">
            <div className="flex items-center gap-3">
              <Wallet className="w-10 h-10" />
              <div>
                <h1 className="text-3xl font-bold">{data.settings.namaMasjid}</h1>
                <p className="text-emerald-100 text-sm">Sistem Pencatatan Zakat</p>
              </div>
            </div>
            
            <div className="flex items-center gap-4">
              {isAuthenticated ? (
                <>
                  <div className="text-right">
                    <p className="font-semibold">{currentUser.nama}</p>
                    <p className="text-xs text-emerald-200">({currentUser.role})</p>
                  </div>
                  <button
                    onClick={onLogout}
                    className="flex items-center gap-2 bg-white/10 px-4 py-2 rounded-xl hover:bg-white/20"
                  >
                    <LogOut className="w-5 h-5" />
                    Logout
                  </button>
                </>
              ) : (
                <button
                  onClick={onShowLogin}
                  className="flex items-center gap-2 bg-white text-emerald-600 px-6 py-2 rounded-xl hover:bg-emerald-50 font-semibold"
                >
                  <Lock className="w-5 h-5" />
                  Login Petugas
                </button>
              )}
            </div>
          </div>
        </div>
      </header>

      <div className="bg-white shadow border-b sticky top-0 z-40">
        <div className="max-w-7xl mx-auto px-4">
          <nav className="flex gap-1 overflow-x-auto">
            {[
              { id: 'dashboard', label: 'Dashboard', icon: BarChart3, public: true },
              { id: 'penerimaan', label: 'Penerimaan', icon: TrendingUp, public: false },
              { id: 'pengeluaran', label: 'Pengeluaran', icon: TrendingDown, public: false },
              { id: 'mustahik', label: 'Mustahik', icon: Users, public: false },
              { id: 'kroscek', label: 'Kroscek Kas', icon: DollarSign, public: false },
              { id: 'laporan', label: 'Laporan', icon: FileText, public: false },
              ...(currentUser?.role === 'Admin' ? [{ id: 'users', label: 'Kelola User', icon: Settings, public: false }] : []),
              ...(isAuthenticated ? [{ id: 'settings', label: 'Pengaturan', icon: Settings, public: false }] : [])
            ].filter(tab => tab.public || isAuthenticated).map(tab => {
              const Icon = tab.icon;
              return (
                <button
                  key={tab.id}
                  onClick={() => setActiveTab(tab.id)}
                  className={`flex items-center gap-2 px-6 py-4 font-medium transition-all whitespace-nowrap ${
                    activeTab === tab.id
                      ? 'text-emerald-700 border-b-3 border-emerald-600 bg-emerald-50'
                      : 'text-gray-600 hover:text-emerald-600'
                  }`}
                >
                  <Icon className="w-5 h-5" />
                  {tab.label}
                </button>
              );
            })}
          </nav>
        </div>
      </div>

      <main className="max-w-7xl mx-auto px-4 py-8">
        {activeTab === 'dashboard' && (
          <DashboardView 
            data={data}
            formatRupiah={formatRupiah}
            totalPenerimaan={totalPenerimaan}
            totalPengeluaran={totalPengeluaran}
            saldo={saldo}
            totalMuzaki={totalMuzaki}
            totalZakatFitrah={totalZakatFitrah}
            totalZakatMaal={totalZakatMaal}
            totalZakatProfesi={totalZakatProfesi}
            totalInfak={totalInfak}
            totalFidyah={totalFidyah}
            totalBeras={totalBeras}
            berasFidyah={berasFidyah}
            berasFitrah={berasFitrah}
            berasSedekah={berasSedekah}
          />
        )}

        {activeTab === 'penerimaan' && isAuthenticated && (
          <DataView
            title="Data Penerimaan"
            data={data.penerimaan}
            onAdd={() => setShowModal({ type: 'penerimaan', data: null })}
            onEdit={(item) => setShowModal({ type: 'penerimaan', data: item })}
            onDelete={(id) => handleDelete('penerimaan', id)}
            formatRupiah={formatRupiah}
            type="penerimaan"
          />
        )}

        {activeTab === 'pengeluaran' && isAuthenticated && (
          <DataView
            title="Data Pengeluaran"
            data={data.pengeluaran}
            onAdd={() => setShowModal({ type: 'pengeluaran', data: null })}
            onEdit={(item) => setShowModal({ type: 'pengeluaran', data: item })}
            onDelete={(id) => handleDelete('pengeluaran', id)}
            formatRupiah={formatRupiah}
            type="pengeluaran"
          />
        )}

        {activeTab === 'mustahik' && isAuthenticated && (
          <MustahikView
            data={data.mustahik}
            onAdd={() => setShowModal({ type: 'mustahik', data: null })}
            onEdit={(item) => setShowModal({ type: 'mustahik', data: item })}
            onDelete={(id) => handleDelete('mustahik', id)}
          />
        )}

        {activeTab === 'kroscek' && isAuthenticated && (
          <KroscekKasView 
            data={data} 
            formatRupiah={formatRupiah}
            kroscekCash={data.kroscekCash}
            kroscekBank={data.kroscekBank}
            onSaveCash={(cashData) => handleSave('kroscekCash', cashData)}
            onSaveBank={(bankAmount) => handleSave('kroscekBank', bankAmount)}
          />
        )}

        {activeTab === 'laporan' && isAuthenticated && (
          <LaporanView data={data} formatRupiah={formatRupiah} />
        )}

        {activeTab === 'users' && currentUser?.role === 'Admin' && (
          <UsersView />
        )}

        {activeTab === 'settings' && isAuthenticated && (
          <SettingsView 
            settings={data.settings}
            onSave={(newSettings) => handleSave('settings', newSettings)}
          />
        )}
      </main>

      {showModal.type === 'penerimaan' && (
        <PenerimaanModal
          data={showModal.data}
          onSave={(newData) => handleSave('penerimaan', newData)}
          onClose={() => setShowModal({ type: null, data: null })}
        />
      )}

      {showModal.type === 'pengeluaran' && (
        <PengeluaranModal
          data={showModal.data}
          mustahik={data.mustahik}
          kategoriList={data.settings.kategoriPengeluaran || ['Distribusi Zakat', 'Program Infak', 'Operasional Masjid', 'Lainnya']}
          onSave={(newData) => handleSave('pengeluaran', newData)}
          onClose={() => setShowModal({ type: null, data: null })}
        />
      )}

      {showModal.type === 'mustahik' && (
        <MustahikModal
          data={showModal.data}
          onSave={(newData) => handleSave('mustahik', newData)}
          onClose={() => setShowModal({ type: null, data: null })}
        />
      )}

      <style>{`
        .border-b-3 {
          border-bottom-width: 3px;
        }
      `}</style>
    </div>
  );
}

function DashboardView({ data, formatRupiah, totalPenerimaan, totalPengeluaran, saldo, totalMuzaki, totalZakatFitrah, totalZakatMaal, totalZakatProfesi, totalInfak, totalFidyah, totalBeras, berasFidyah, berasFitrah, berasSedekah }) {
  const targetZakat = data.settings.targetZakatFitrah || 0;
  const progress = targetZakat > 0 ? (totalZakatFitrah / targetZakat) * 100 : 0;

  return (
    <div className="space-y-6">
      <h2 className="text-3xl font-bold text-gray-800">Dashboard</h2>

      {targetZakat > 0 && (
        <div className="bg-gradient-to-br from-blue-600 to-cyan-600 rounded-2xl shadow-xl p-6 text-white">
          <div className="flex items-center gap-3 mb-4">
            <Target className="w-8 h-8" />
            <h3 className="text-2xl font-bold">Target Zakat Fitrah</h3>
          </div>
          <div className="space-y-3">
            <div className="flex justify-between text-sm">
              <span>Progress: {progress.toFixed(1)}%</span>
              <span>{formatRupiah(totalZakatFitrah)} / {formatRupiah(targetZakat)}</span>
            </div>
            <div className="w-full bg-white/20 rounded-full h-4">
              <div 
                className="bg-white h-full rounded-full transition-all"
                style={{ width: `${Math.min(progress, 100)}%` }}
              />
            </div>
          </div>
        </div>
      )}
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <StatCard title="Total Penerimaan" value={formatRupiah(totalPenerimaan)} icon={TrendingUp} color="emerald" />
        <StatCard title="Total Pengeluaran" value={formatRupiah(totalPengeluaran)} icon={TrendingDown} color="red" />
        <StatCard title="Saldo Saat Ini" value={formatRupiah(saldo)} icon={Wallet} color="blue" />
        <StatCard title="Total Muzaki" value={totalMuzaki} icon={Users} color="purple" />
      </div>

      <div className="bg-white rounded-2xl shadow-lg p-6 border">
        <h3 className="text-xl font-bold mb-4 flex items-center gap-2">
          <DollarSign className="w-6 h-6 text-emerald-600" />
          Penerimaan Uang
        </h3>
        <div className="grid grid-cols-2 md:grid-cols-5 gap-4">
          <BreakdownCard title="Zakat Fitrah" amount={formatRupiah(totalZakatFitrah)} color="emerald" />
          <BreakdownCard title="Zakat Maal" amount={formatRupiah(totalZakatMaal)} color="teal" />
          <BreakdownCard title="Zakat Profesi" amount={formatRupiah(totalZakatProfesi)} color="cyan" />
          <BreakdownCard title="Infak" amount={formatRupiah(totalInfak)} color="blue" />
          <BreakdownCard title="Fidyah" amount={formatRupiah(totalFidyah)} color="indigo" />
        </div>
      </div>

      {totalBeras > 0 && (
        <div className="bg-white rounded-2xl shadow-lg p-6 border">
          <h3 className="text-xl font-bold mb-4">Penerimaan Beras</h3>
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
            <BerasCard title="Fidyah" kg={berasFidyah} color="amber" />
            <BerasCard title="Fitrah" kg={berasFitrah} color="orange" />
            <BerasCard title="Sedekah" kg={berasSedekah} color="yellow" />
            <BerasCard title="Total" kg={totalBeras} color="emerald" />
          </div>
        </div>
      )}

      <div className="bg-white rounded-2xl shadow-lg p-6 border">
        <h3 className="text-xl font-bold mb-4">Informasi Rekening</h3>
        <div className="bg-blue-50 p-4 rounded-lg space-y-2">
          <p><strong>Bank:</strong> {data.settings.namaBank || 'Bank Syariah Indonesia'}</p>
          <p><strong>Kode Bank:</strong> {data.settings.kodeBank || '451'}</p>
          <p><strong>No. Rekening:</strong> {data.settings.noRekening || '7019291698'}</p>
          <p><strong>Atas Nama:</strong> {data.settings.atasNama || 'Eko Andri QQ BAITUL HIKMAH'}</p>
          <p><strong>Konsultasi:</strong> <a href={`https://wa.me/${data.settings.noWAKonsultasi || '6289601705274'}`} target="_blank" className="text-blue-600 hover:underline">wa.me/{data.settings.noWAKonsultasi || '6289601705274'}</a></p>
        </div>
      </div>

      <div className="bg-white rounded-2xl shadow-lg p-6 border">
        <h3 className="text-xl font-bold mb-4">5 Transaksi Terbaru</h3>
        <div className="space-y-3">
          {[...data.penerimaan.slice(0, 3), ...data.pengeluaran.slice(0, 2)]
            .sort((a, b) => b.id - a.id)
            .slice(0, 5)
            .map(item => (
              <div key={item.id} className="flex items-center justify-between p-4 bg-gray-50 rounded-xl hover:bg-emerald-50 transition-colors">
                <div className="flex items-center gap-4">
                  <div className={`p-3 rounded-full ${item.jenis ? 'bg-emerald-100' : 'bg-red-100'}`}>
                    {item.jenis ? <TrendingUp className="w-5 h-5 text-emerald-600" /> : <TrendingDown className="w-5 h-5 text-red-600" />}
                  </div>
                  <div>
                    <p className="font-semibold text-gray-800">
                      {item.jenis || 'Pengeluaran'} - {item.donatur || item.penerima}
                    </p>
                    <p className="text-sm text-gray-500">{item.tanggal} • {item.petugas || 'System'}</p>
                  </div>
                </div>
                <div className="text-right">
                  <p className={`font-bold text-lg ${item.jenis ? 'text-emerald-600' : 'text-red-600'}`}>
                    {item.jenis ? '+' : '-'}{formatRupiah(item.jumlah)}
                  </p>
                  {item.jumlahBeras > 0 && (
                    <p className="text-sm text-gray-500">{item.jumlahBeras} Kg</p>
                  )}
                </div>
              </div>
            ))}
        </div>
        {[...data.penerimaan, ...data.pengeluaran].length === 0 && (
          <div className="text-center py-8 text-gray-500">Belum ada transaksi</div>
        )}
      </div>
    </div>
  );
}

function StatCard({ title, value, icon: Icon, color }) {
  const colors = {
    emerald: 'from-emerald-600 to-teal-600',
    red: 'from-red-600 to-rose-600',
    blue: 'from-blue-600 to-cyan-600',
    purple: 'from-purple-600 to-indigo-600'
  };

  return (
    <div className="bg-white rounded-2xl shadow-lg p-6 border hover:shadow-xl transition-shadow">
      <div className="flex items-center justify-between mb-4">
        <p className="text-gray-600 font-medium text-sm">{title}</p>
        <div className={`p-3 rounded-full bg-gradient-to-br ${colors[color]}`}>
          <Icon className="w-6 h-6 text-white" />
        </div>
      </div>
      <p className="text-3xl font-bold text-gray-800">{value}</p>
    </div>
  );
}

function BreakdownCard({ title, amount, color }) {
  const colors = {
    emerald: 'from-emerald-600 to-teal-600',
    teal: 'from-teal-600 to-cyan-600',
    cyan: 'from-cyan-600 to-blue-600',
    blue: 'from-blue-600 to-indigo-600',
    indigo: 'from-indigo-600 to-purple-600'
  };

  return (
    <div className={`bg-gradient-to-br ${colors[color]} rounded-xl shadow-lg p-4 text-white`}>
      <p className="text-white/90 text-xs mb-2">{title}</p>
      <p className="text-xl font-bold">{amount}</p>
    </div>
  );
}

function BerasCard({ title, kg, color }) {
  const colors = {
    amber: 'from-amber-500 to-orange-500',
    orange: 'from-orange-500 to-red-500',
    yellow: 'from-yellow-500 to-amber-500',
    emerald: 'from-emerald-600 to-teal-600'
  };

  return (
    <div className={`bg-gradient-to-br ${colors[color]} rounded-xl shadow-lg p-4 text-white`}>
      <p className="text-white/90 text-xs mb-2">{title}</p>
      <p className="text-xl font-bold">{kg.toFixed(2)} Kg</p>
    </div>
  );
}

function DataView({ title, data, onAdd, onEdit, onDelete, formatRupiah, type }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [detailModal, setDetailModal] = useState(null);

  const filteredData = data.filter(item => {
    const searchLower = searchTerm.toLowerCase();
    if (type === 'penerimaan') {
      return item.donatur?.toLowerCase().includes(searchLower) ||
             item.jenis?.toLowerCase().includes(searchLower) ||
             item.lokasi?.toLowerCase().includes(searchLower);
    } else {
      return item.penerima?.toLowerCase().includes(searchLower) ||
             item.kategori?.toLowerCase().includes(searchLower);
    }
  });

  return (
    <div className="space-y-6">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
        <h2 className="text-3xl font-bold">{title}</h2>
        <div className="flex flex-col md:flex-row gap-3 w-full md:w-auto">
          <input
            type="text"
            placeholder="🔍 Cari data..."
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            className="px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-transparent w-full md:w-64"
          />
          <button
            onClick={onAdd}
            className="flex items-center justify-center gap-2 bg-gradient-to-r from-emerald-600 to-teal-600 text-white px-6 py-3 rounded-xl hover:from-emerald-700 hover:to-teal-700 font-semibold shadow-lg whitespace-nowrap"
          >
            <PlusCircle className="w-5 h-5" />
            Tambah {type === 'penerimaan' ? 'Penerimaan' : 'Pengeluaran'}
          </button>
        </div>
      </div>

      <div className="bg-white rounded-2xl shadow-lg overflow-hidden">
        <div className="overflow-x-auto">
          <table className="w-full">
            <thead className="bg-gradient-to-r from-emerald-600 to-teal-600 text-white">
              <tr>
                <th className="px-6 py-4 text-left">Tanggal</th>
                <th className="px-6 py-4 text-left">{type === 'penerimaan' ? 'Jenis' : 'Kategori'}</th>
                <th className="px-6 py-4 text-left">{type === 'penerimaan' ? 'Donatur' : 'Penerima'}</th>
                {type === 'penerimaan' && <th className="px-6 py-4 text-left">Lokasi</th>}
                <th className="px-6 py-4 text-right">Uang</th>
                {type === 'penerimaan' && <th className="px-6 py-4 text-right">Beras</th>}
                <th className="px-6 py-4 text-left">Petugas</th>
                <th className="px-6 py-4 text-center">Aksi</th>
              </tr>
            </thead>
            <tbody>
              {filteredData.map(item => (
                <tr key={item.id} className="border-b hover:bg-emerald-50">
                  <td className="px-6 py-4">{item.tanggal}</td>
                  <td className="px-6 py-4">
                    <span className={`px-3 py-1 rounded-full text-xs font-medium ${
                      type === 'penerimaan' 
                        ? (item.jenis === 'Zakat Fitrah' ? 'bg-emerald-100 text-emerald-700' :
                           item.jenis === 'Zakat Maal' ? 'bg-teal-100 text-teal-700' :
                           item.jenis === 'Zakat Profesi' ? 'bg-cyan-100 text-cyan-700' :
                           item.jenis === 'Infak' ? 'bg-blue-100 text-blue-700' :
                           'bg-indigo-100 text-indigo-700')
                        : 'bg-gray-100 text-gray-700'
                    }`}>
                      {item.jenis || item.kategori}
                    </span>
                  </td>
                  <td className="px-6 py-4">
                    <div>
                      <p className="font-medium">{item.donatur || item.penerima}</p>
                      {item.anggotaKeluarga && item.anggotaKeluarga.length > 0 && (
                        <p className="text-xs text-gray-500">👥 {item.anggotaKeluarga.length} anggota</p>
                      )}
                    </div>
                  </td>
                  {type === 'penerimaan' && (
                    <td className="px-6 py-4">
                      <span className="flex items-center gap-1 text-sm text-gray-600">
                        <MapPin className="w-4 h-4" />
                        {item.lokasi}
                      </span>
                    </td>
                  )}
                  <td className="px-6 py-4 text-right font-bold text-emerald-600">
                    {formatRupiah(item.jumlah || 0)}
                  </td>
                  {type === 'penerimaan' && (
                    <td className="px-6 py-4 text-right">
                      {item.jumlahBeras > 0 ? (
                        <span className="font-medium text-amber-600">
                          {item.jumlahBeras} Kg<br/>
                          <span className="text-xs text-gray-500">({item.jenisBeras})</span>
                        </span>
                      ) : (
                        <span className="text-gray-400">-</span>
                      )}
                    </td>
                  )}
                  <td className="px-6 py-4 text-sm text-gray-600">{item.petugas || 'System'}</td>
                  <td className="px-6 py-4 text-center">
                    <div className="flex gap-2 justify-center">
                      <button 
                        onClick={() => setDetailModal(item)} 
                        className="text-blue-600 hover:text-blue-800 p-2 rounded-lg hover:bg-blue-50"
                        title="Lihat Detail"
                      >
                        <Eye className="w-5 h-5" />
                      </button>
                      <button 
                        onClick={() => onEdit(item)} 
                        className="text-emerald-600 hover:text-emerald-800 p-2 rounded-lg hover:bg-emerald-50"
                        title="Edit"
                      >
                        <Edit2 className="w-5 h-5" />
                      </button>
                      <button 
                        onClick={() => onDelete(item.id)} 
                        className="text-red-600 hover:text-red-800 p-2 rounded-lg hover:bg-red-50"
                        title="Hapus"
                      >
                        <X className="w-5 h-5" />
                      </button>
                    </div>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
        {filteredData.length === 0 && (
          <div className="text-center py-12 text-gray-500">
            {searchTerm ? 'Tidak ada data yang cocok dengan pencarian' : 'Belum ada data'}
          </div>
        )}
      </div>

      {/* Detail Modal */}
      {detailModal && (
        <div className="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
          <div className="bg-white rounded-2xl shadow-2xl max-w-2xl w-full p-8 max-h-[90vh] overflow-y-auto">
            <div className="flex justify-between items-center mb-6">
              <h3 className="text-2xl font-bold">Detail {type === 'penerimaan' ? 'Penerimaan' : 'Pengeluaran'}</h3>
              <button onClick={() => setDetailModal(null)} className="text-gray-400 hover:text-gray-600">
                <X className="w-6 h-6" />
              </button>
            </div>

            <div className="space-y-4">
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <p className="text-sm text-gray-500">Tanggal</p>
                  <p className="font-semibold text-gray-800">{detailModal.tanggal}</p>
                </div>
                <div>
                  <p className="text-sm text-gray-500">{type === 'penerimaan' ? 'Jenis' : 'Kategori'}</p>
                  <p className="font-semibold text-gray-800">{detailModal.jenis || detailModal.kategori}</p>
                </div>
              </div>

              <div className="border-t pt-4">
                <p className="text-sm text-gray-500 mb-2">{type === 'penerimaan' ? 'Donatur' : 'Penerima'}</p>
                <p className="font-semibold text-gray-800 text-lg">{detailModal.donatur || detailModal.penerima}</p>
              </div>

              {detailModal.alamat && (
                <div>
                  <p className="text-sm text-gray-500">Alamat</p>
                  <p className="text-gray-700">{detailModal.alamat}</p>
                </div>
              )}

              {detailModal.noHP && (
                <div>
                  <p className="text-sm text-gray-500">No HP/WhatsApp</p>
                  <p className="text-gray-700">{detailModal.noHP}</p>
                </div>
              )}

              {detailModal.jumlahKeluarga && (
                <div>
                  <p className="text-sm text-gray-500">Jumlah Keluarga</p>
                  <p className="text-gray-700">{detailModal.jumlahKeluarga} orang</p>
                </div>
              )}

              {detailModal.anggotaKeluarga && detailModal.anggotaKeluarga.length > 0 && (
                <div>
                  <p className="text-sm text-gray-500 mb-2">Anggota Keluarga</p>
                  <div className="bg-gray-50 p-4 rounded-lg space-y-1">
                    {detailModal.anggotaKeluarga.map((nama, i) => (
                      <p key={i} className="text-gray-700">{i + 1}. {nama}</p>
                    ))}
                  </div>
                </div>
              )}

              {detailModal.lokasi && (
                <div>
                  <p className="text-sm text-gray-500">Lokasi Counter</p>
                  <p className="text-gray-700">{detailModal.lokasi}</p>
                </div>
              )}

              {detailModal.metodePembayaran && (
                <div>
                  <p className="text-sm text-gray-500">Metode Pembayaran</p>
                  <p className="text-gray-700">{detailModal.metodePembayaran}</p>
                </div>
              )}

              <div className="bg-emerald-50 p-4 rounded-lg">
                <p className="text-sm text-emerald-700 mb-2">Jumlah</p>
                <div className="space-y-2">
                  <p className="text-lg font-bold text-emerald-700">💰 Uang: {formatRupiah(detailModal.jumlah || 0)}</p>
                  {detailModal.jumlahBeras > 0 && (
                    <p className="text-lg font-bold text-amber-700">🍚 Beras: {detailModal.jumlahBeras} Kg ({detailModal.jenisBeras})</p>
                  )}
                </div>
              </div>

              {detailModal.keterangan && (
                <div>
                  <p className="text-sm text-gray-500">Keterangan</p>
                  <p className="text-gray-700">{detailModal.keterangan}</p>
                </div>
              )}

              <div className="text-sm text-gray-500 border-t pt-4">
                <p>Diinput oleh: <span className="font-medium text-gray-700">{detailModal.petugas || 'System'}</span></p>
              </div>
            </div>

            {/* Action Buttons */}
            <div className="flex gap-3 mt-6 pt-6 border-t">
              {detailModal.noHP && (
                <button
                  onClick={() => {
                    const formatRupiah = (amount) => {
                      return new Intl.NumberFormat('id-ID', {
                        style: 'currency',
                        currency: 'IDR',
                        minimumFractionDigits: 0
                      }).format(amount);
                    };

                    let message = `📢 *Konfirmasi Penerimaan Zakat*\n\n`;
                    message += `👤 *Nama:* ${detailModal.donatur}\n`;
                    if (detailModal.alamat) message += `📍 *Alamat:* ${detailModal.alamat}\n`;
                    message += `📱 *No HP:* ${detailModal.noHP}\n`;
                    if (detailModal.jumlahKeluarga) message += `👥 *Jumlah Keluarga:* ${detailModal.jumlahKeluarga} orang\n`;
                    
                    if (detailModal.anggotaKeluarga && detailModal.anggotaKeluarga.length > 0) {
                      message += `\n👨‍👩‍👧‍👦 *Anggota Keluarga:*\n`;
                      message += detailModal.anggotaKeluarga.map((nama, i) => `${i + 1}. ${nama}`).join('\n');
                      message += '\n';
                    }
                    
                    message += `\n💵 *Metode:* ${detailModal.metodePembayaran}\n`;
                    message += `📍 *Lokasi:* ${detailModal.lokasi}\n\n`;
                    message += `💰 *${detailModal.jenis}:* ${formatRupiah(detailModal.jumlah || 0)}`;
                    
                    if (detailModal.jumlahBeras > 0) {
                      message += ` + ${detailModal.jumlahBeras} Kg Beras (${detailModal.jenisBeras})`;
                    }
                    
                    message += `\n\n🙏 Jazakumullahu khairan`;

                    const phoneNumber = detailModal.noHP.replace(/\D/g, '');
                    const whatsappURL = `https://wa.me/${phoneNumber}?text=${encodeURIComponent(message)}`;
                    window.open(whatsappURL, '_blank');
                  }}
                  className="flex-1 flex items-center justify-center gap-2 bg-green-600 text-white px-6 py-3 rounded-xl hover:bg-green-700 font-semibold shadow-lg"
                >
                  <svg className="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893a11.821 11.821 0 00-3.48-8.413Z"/>
                  </svg>
                  Kirim WA
                </button>
              )}
              
              <button
                onClick={() => {
                  // Create canvas untuk generate gambar
                  const canvas = document.createElement('canvas');
                  const ctx = canvas.getContext('2d');
                  
                  // Set ukuran canvas (58mm = ~220px untuk 96 DPI)
                  const width = 400; // Lebar dalam pixel
                  const padding = 20;
                  let yPos = padding;
                  
                  // Fungsi untuk mengukur tinggi text
                  const measureText = (text, fontSize, maxWidth) => {
                    ctx.font = `${fontSize}px monospace`;
                    const words = text.split(' ');
                    const lines = [];
                    let currentLine = '';
                    
                    words.forEach(word => {
                      const testLine = currentLine + (currentLine ? ' ' : '') + word;
                      const metrics = ctx.measureText(testLine);
                      if (metrics.width > maxWidth && currentLine) {
                        lines.push(currentLine);
                        currentLine = word;
                      } else {
                        currentLine = testLine;
                      }
                    });
                    if (currentLine) lines.push(currentLine);
                    return lines;
                  };
                  
                  // Hitung tinggi total yang dibutuhkan
                  let totalHeight = padding * 2;
                  totalHeight += 80; // Header
                  totalHeight += 20; // Line
                  totalHeight += 25; // Nama
                  if (detailModal.alamat) {
                    const alamatLines = measureText(detailModal.alamat, 14, width - padding * 2);
                    totalHeight += alamatLines.length * 20;
                  }
                  if (detailModal.noHP) totalHeight += 20;
                  totalHeight += 30; // Space
                  totalHeight += 20; // Line
                  totalHeight += 80; // Detail transaksi (4 baris)
                  totalHeight += 20; // Line
                  totalHeight += 30; // Space
                  totalHeight += 60; // Total
                  if (detailModal.jumlahBeras > 0) totalHeight += 25;
                  totalHeight += 30; // Space
                  totalHeight += 20; // Line
                  totalHeight += 60; // Footer
                  totalHeight += 20; // Line
                  totalHeight += 30; // Petugas
                  
                  canvas.width = width;
                  canvas.height = totalHeight;
                  
                  // Background putih
                  ctx.fillStyle = '#ffffff';
                  ctx.fillRect(0, 0, width, totalHeight);
                  
                  // Set default style
                  ctx.fillStyle = '#000000';
                  ctx.textAlign = 'center';
                  
                  // Header - KWITANSI ZAKAT
                  ctx.font = 'bold 20px monospace';
                  ctx.fillText('KWITANSI ZAKAT', width / 2, yPos);
                  yPos += 25;
                  
                  // Nama Masjid
                  ctx.font = '16px monospace';
                  ctx.fillText(data.settings.namaMasjid, width / 2, yPos);
                  yPos += 30;
                  
                  // Line separator
                  ctx.strokeStyle = '#000000';
                  ctx.lineWidth = 2;
                  ctx.beginPath();
                  ctx.moveTo(padding, yPos);
                  ctx.lineTo(width - padding, yPos);
                  ctx.stroke();
                  yPos += 25;
                  
                  // Data Donatur - align left
                  ctx.textAlign = 'left';
                  ctx.font = '14px monospace';
                  
                  ctx.fillText(`Nama: ${detailModal.donatur}`, padding, yPos);
                  yPos += 20;
                  
                  if (detailModal.alamat) {
                    const alamatLines = measureText(detailModal.alamat, 14, width - padding * 2);
                    ctx.fillText(`Alamat: ${alamatLines[0]}`, padding, yPos);
                    yPos += 20;
                    for (let i = 1; i < alamatLines.length; i++) {
                      ctx.fillText(`        ${alamatLines[i]}`, padding, yPos);
                      yPos += 20;
                    }
                  }
                  
                  if (detailModal.noHP) {
                    ctx.fillText(`No HP: ${detailModal.noHP}`, padding, yPos);
                    yPos += 20;
                  }
                  
                  yPos += 10;
                  
                  // Dashed line
                  ctx.setLineDash([5, 3]);
                  ctx.lineWidth = 1;
                  ctx.beginPath();
                  ctx.moveTo(padding, yPos);
                  ctx.lineTo(width - padding, yPos);
                  ctx.stroke();
                  ctx.setLineDash([]);
                  yPos += 20;
                  
                  // Detail Transaksi
                  ctx.fillText(`Tanggal: ${detailModal.tanggal}`, padding, yPos);
                  yPos += 20;
                  ctx.fillText(`Jenis: ${detailModal.jenis}`, padding, yPos);
                  yPos += 20;
                  ctx.fillText(`Lokasi: ${detailModal.lokasi}`, padding, yPos);
                  yPos += 20;
                  ctx.fillText(`Metode: ${detailModal.metodePembayaran}`, padding, yPos);
                  yPos += 20;
                  
                  // Dashed line
                  ctx.setLineDash([5, 3]);
                  ctx.beginPath();
                  ctx.moveTo(padding, yPos);
                  ctx.lineTo(width - padding, yPos);
                  ctx.stroke();
                  ctx.setLineDash([]);
                  yPos += 30;
                  
                  // TOTAL - center and bold
                  ctx.textAlign = 'center';
                  ctx.font = 'bold 18px monospace';
                  ctx.fillText('TOTAL: Rp ' + (detailModal.jumlah || 0).toLocaleString('id-ID'), width / 2, yPos);
                  yPos += 25;
                  
                  if (detailModal.jumlahBeras > 0) {
                    ctx.font = '14px monospace';
                    ctx.fillText(`Beras: ${detailModal.jumlahBeras} Kg (${detailModal.jenisBeras})`, width / 2, yPos);
                    yPos += 25;
                  }
                  
                  yPos += 10;
                  
                  // Line separator
                  ctx.lineWidth = 2;
                  ctx.beginPath();
                  ctx.moveTo(padding, yPos);
                  ctx.lineTo(width - padding, yPos);
                  ctx.stroke();
                  yPos += 25;
                  
                  // Footer
                  ctx.font = '14px monospace';
                  ctx.fillText('Jazakumullahu Khairan', width / 2, yPos);
                  yPos += 20;
                  ctx.fillText('Semoga Allah memberkahi', width / 2, yPos);
                  yPos += 30;
                  
                  // Line separator
                  ctx.lineWidth = 2;
                  ctx.beginPath();
                  ctx.moveTo(padding, yPos);
                  ctx.lineTo(width - padding, yPos);
                  ctx.stroke();
                  yPos += 25;
                  
                  // Petugas
                  ctx.font = '12px monospace';
                  ctx.fillText(`Petugas: ${detailModal.petugas || 'System'}`, width / 2, yPos);
                  
                  // Convert canvas to blob and download
                  canvas.toBlob((blob) => {
                    const url = URL.createObjectURL(blob);
                    const a = document.createElement('a');
                    a.href = url;
                    a.download = `Kwitansi_${detailModal.donatur.replace(/\s+/g, '_')}_${detailModal.tanggal}.png`;
                    document.body.appendChild(a);
                    a.click();
                    document.body.removeChild(a);
                    URL.revokeObjectURL(url);
                  }, 'image/png');
                }}
                className="flex-1 flex items-center justify-center gap-2 bg-blue-600 text-white px-6 py-3 rounded-xl hover:bg-blue-700 font-semibold shadow-lg"
              >
                <Download className="w-5 h-5" />
                Download Kwitansi
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

function MustahikView({ data, onAdd, onEdit, onDelete }) {
  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h2 className="text-3xl font-bold">Data Mustahik</h2>
        <button
          onClick={onAdd}
          className="flex items-center gap-2 bg-gradient-to-r from-purple-600 to-indigo-600 text-white px-6 py-3 rounded-xl hover:from-purple-700 hover:to-indigo-700 font-semibold shadow-lg"
        >
          <PlusCircle className="w-5 h-5" />
          Tambah Mustahik
        </button>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {data.map(person => (
          <div key={person.id} className="bg-white rounded-2xl shadow-lg p-6 border hover:shadow-xl transition-shadow">
            <div className="flex justify-between items-start mb-4">
              <div>
                <h3 className="font-bold text-lg text-gray-800">{person.nama}</h3>
                <span className="text-xs px-2 py-1 bg-emerald-100 text-emerald-700 rounded-full">
                  {person.status}
                </span>
              </div>
              <div className="flex gap-2">
                <button onClick={() => onEdit(person)} className="text-blue-600 hover:text-blue-800 p-2 rounded-lg hover:bg-blue-50">
                  <Edit2 className="w-5 h-5" />
                </button>
                <button onClick={() => onDelete(person.id)} className="text-red-600 hover:text-red-800 p-2 rounded-lg hover:bg-red-50">
                  <X className="w-5 h-5" />
                </button>
              </div>
            </div>
            <div className="space-y-2 text-sm text-gray-600">
              <p><strong>Alamat:</strong> {person.alamat}</p>
              <p><strong>No. Telp:</strong> {person.noTelp}</p>
              <p><strong>Kategori:</strong> {person.kategori}</p>
            </div>
          </div>
        ))}
      </div>

      {data.length === 0 && (
        <div className="bg-white rounded-2xl shadow-lg p-12 text-center text-gray-500">
          Belum ada data mustahik
        </div>
      )}
    </div>
  );
}

function KroscekKasView({ data, formatRupiah, kroscekCash, kroscekBank, onSaveCash, onSaveBank }) {
  const [uangCash, setUangCash] = useState(kroscekCash || {
    100000: 0,
    50000: 0,
    20000: 0,
    10000: 0,
    5000: 0,
    2000: 0,
    1000: 0,
    500: 0,
    200: 0,
    100: 0
  });
  const [uangDiBank, setUangDiBank] = useState(kroscekBank || 0);

  // Update local state ketika props berubah
  useEffect(() => {
    if (kroscekCash) setUangCash(kroscekCash);
    if (kroscekBank !== undefined) setUangDiBank(kroscekBank);
  }, [kroscekCash, kroscekBank]);

  // Simpan uang cash setiap kali berubah
  const handleCashChange = (nominal, value) => {
    const newCash = {...uangCash, [nominal]: value === '' ? 0 : parseInt(value) || 0};
    setUangCash(newCash);
    onSaveCash(newCash);
  };

  // Simpan uang bank setiap kali berubah
  const handleBankChange = (value) => {
    const newBank = value === '' ? 0 : parseFloat(value) || 0;
    setUangDiBank(newBank);
    onSaveBank(newBank);
  };

  // Hitung total uang cash
  const totalUangCash = Object.entries(uangCash).reduce((sum, [nominal, pcs]) => 
    sum + (parseInt(nominal) * pcs), 0
  );

  // Data dari sistem
  const totalPenerimaanData = data.penerimaan.reduce((sum, p) => sum + (p.jumlah || 0), 0);
  const totalPengeluaranData = data.pengeluaran.reduce((sum, p) => sum + (p.jumlah || 0), 0);
  
  // Uang Non Tunai dari data penerimaan (Transfer + QRIS)
  const uangNonTunaiData = data.penerimaan
    .filter(p => p.metodePembayaran === 'Transfer' || p.metodePembayaran === 'QRIS')
    .reduce((sum, p) => sum + (p.jumlah || 0), 0);

  // Rumus: (Uang Non Tunai + Uang Cash + Uang di Bank) - (Data Pemasukan - Pengeluaran)
  const saldoSistem = totalPenerimaanData - totalPengeluaranData;
  const saldoFisik = uangNonTunaiData + totalUangCash + uangDiBank;
  const selisih = saldoFisik - saldoSistem;

  return (
    <div className="space-y-6">
      <h2 className="text-3xl font-bold">Kroscek Kas</h2>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* KOLOM 1: Uang Cash */}
        <div className="bg-white rounded-2xl shadow-lg p-6 border lg:col-span-2">
          <h3 className="text-xl font-bold mb-4 text-emerald-700 flex items-center gap-2">
            <DollarSign className="w-6 h-6" />
            💵 Uang Cash (Fisik)
          </h3>
          <div className="space-y-2">
            <div className="grid grid-cols-4 gap-2 font-bold text-sm text-gray-600 border-b pb-2">
              <div>Nominal</div>
              <div className="text-center">Pcs</div>
              <div className="text-right col-span-2">Total</div>
            </div>
            
            {Object.entries(uangCash).map(([nominal, pcs]) => {
              const total = parseInt(nominal) * pcs;
              return (
                <div key={nominal} className="grid grid-cols-4 gap-2 items-center hover:bg-emerald-50 p-2 rounded-lg">
                  <div className="font-semibold text-gray-700">
                    {formatRupiah(parseInt(nominal))}
                  </div>
                  <input
                    type="number"
                    min="0"
                    value={pcs === 0 ? '' : pcs}
                    onChange={(e) => handleCashChange(nominal, e.target.value)}
                    className="px-2 py-2 border rounded-lg focus:ring-2 focus:ring-emerald-500 text-center font-medium"
                    placeholder="0"
                  />
                  <div className="text-right font-medium text-emerald-600 col-span-2">
                    {formatRupiah(total)}
                  </div>
                </div>
              );
            })}
            
            <div className="border-t-2 border-emerald-600 pt-3 mt-3">
              <div className="grid grid-cols-4 gap-2 items-center bg-emerald-50 p-3 rounded-lg">
                <div className="font-bold text-lg col-span-2">Total Uang Cash:</div>
                <div className="text-right font-bold text-2xl text-emerald-600 col-span-2">
                  {formatRupiah(totalUangCash)}
                </div>
              </div>
            </div>
          </div>
        </div>

        {/* KOLOM 2: Perhitungan dari Sistem */}
        <div className="space-y-6">
          {/* Uang di Bank */}
          <div className="bg-white rounded-2xl shadow-lg p-6 border">
            <h3 className="text-xl font-bold mb-4 text-indigo-700">🏦 Uang di Bank</h3>
            <div className="space-y-3">
              <label className="block text-sm font-semibold">Jumlah Uang di Bank (Manual)</label>
              <input
                type="number"
                value={uangDiBank === 0 ? '' : uangDiBank}
                onChange={(e) => handleBankChange(e.target.value)}
                className="w-full px-4 py-3 border-2 rounded-xl focus:ring-2 focus:ring-indigo-500 text-2xl font-bold text-indigo-600"
                placeholder="0"
              />
              <div className="bg-indigo-50 p-3 rounded-lg border-2 border-indigo-200 mt-2">
                <p className="text-xl font-bold text-indigo-600">{formatRupiah(uangDiBank)}</p>
                <p className="text-xs text-gray-600 mt-1">Input manual saldo bank saat ini</p>
              </div>
            </div>
          </div>

          {/* Uang Non Tunai dari Sistem */}
          <div className="bg-white rounded-2xl shadow-lg p-6 border">
            <h3 className="text-xl font-bold mb-4 text-blue-700">💳 Uang Non Tunai (Sistem)</h3>
            <div className="space-y-3">
              <label className="block text-sm font-semibold">Transfer + QRIS (Otomatis)</label>
              <div className="bg-blue-50 p-4 rounded-lg border-2 border-blue-300">
                <p className="text-3xl font-bold text-blue-600">{formatRupiah(uangNonTunaiData)}</p>
                <p className="text-xs text-gray-600 mt-2">Data dari pencatatan penerimaan</p>
              </div>
            </div>
          </div>

          {/* Pengeluaran dari Sistem */}
          <div className="bg-white rounded-2xl shadow-lg p-6 border">
            <h3 className="text-xl font-bold mb-4 text-red-700">💸 Pengeluaran (Sistem)</h3>
            <div className="space-y-3">
              <label className="block text-sm font-semibold">Total Pengeluaran (Otomatis)</label>
              <div className="bg-red-50 p-4 rounded-lg border-2 border-red-300">
                <p className="text-3xl font-bold text-red-600">{formatRupiah(totalPengeluaranData)}</p>
                <p className="text-xs text-gray-600 mt-2">Data dari pencatatan pengeluaran</p>
              </div>
            </div>
          </div>

          {/* Saldo Sistem */}
          <div className="bg-white rounded-2xl shadow-lg p-6 border">
            <h3 className="text-xl font-bold mb-4 text-emerald-700">💰 Saldo Sistem</h3>
            <div className="bg-emerald-50 p-4 rounded-lg border-2 border-emerald-300">
              <p className="text-sm text-gray-600 mb-2">Pemasukan - Pengeluaran</p>
              <p className="text-3xl font-bold text-emerald-600">{formatRupiah(saldoSistem)}</p>
            </div>
          </div>
        </div>
      </div>

      {/* PERHITUNGAN AKHIR */}
      <div className="bg-gradient-to-br from-purple-50 to-pink-50 rounded-2xl shadow-xl p-8 border-2 border-purple-200">
        <h3 className="text-2xl font-bold mb-6 text-purple-800 text-center">📊 Perhitungan Kroscek</h3>
        
        <div className="bg-white rounded-xl p-6 space-y-4 shadow-lg">
          <div className="flex justify-between items-center py-3 border-b-2">
            <span className="font-semibold text-gray-700">Uang Cash (Fisik)</span>
            <span className="font-bold text-lg text-emerald-600">{formatRupiah(totalUangCash)}</span>
          </div>
          
          <div className="flex justify-between items-center py-3 border-b-2">
            <span className="font-semibold text-gray-700">Uang di Bank (+)</span>
            <span className="font-bold text-lg text-indigo-600">{formatRupiah(uangDiBank)}</span>
          </div>
          
          <div className="flex justify-between items-center py-3 border-b-2">
            <span className="font-semibold text-gray-700">Uang Non Tunai (Sistem) (+)</span>
            <span className="font-bold text-lg text-blue-600">{formatRupiah(uangNonTunaiData)}</span>
          </div>
          
          <div className="flex justify-between items-center py-3 bg-purple-50 px-4 rounded-lg border-2 border-purple-300">
            <span className="font-bold text-gray-800">Saldo Fisik (Cash + Bank + Non Tunai)</span>
            <span className="font-bold text-xl text-purple-600">
              {formatRupiah(saldoFisik)}
            </span>
          </div>
          
          <div className="border-t-2 border-gray-300 pt-4 mt-4"></div>
          
          <div className="flex justify-between items-center py-3 border-b-2">
            <span className="font-semibold text-gray-700">Total Pemasukan (Sistem)</span>
            <span className="font-bold text-lg text-emerald-600">{formatRupiah(totalPenerimaanData)}</span>
          </div>
          
          <div className="flex justify-between items-center py-3 border-b-2">
            <span className="font-semibold text-gray-700">Total Pengeluaran (Sistem) (-)</span>
            <span className="font-bold text-lg text-red-600">{formatRupiah(totalPengeluaranData)}</span>
          </div>
          
          <div className="flex justify-between items-center py-3 bg-emerald-50 px-4 rounded-lg border-2 border-emerald-300">
            <span className="font-bold text-gray-800">Saldo Sistem</span>
            <span className="font-bold text-xl text-emerald-600">{formatRupiah(saldoSistem)}</span>
          </div>
          
          <div className="border-t-4 border-purple-400 pt-4 mt-4">
            <div className={`p-6 rounded-xl ${
              selisih === 0 ? 'bg-green-100 border-4 border-green-400' : 
              selisih > 0 ? 'bg-blue-100 border-4 border-blue-400' : 
              'bg-red-100 border-4 border-red-400'
            }`}>
              <p className="text-sm font-semibold text-gray-600 text-center mb-2">SELISIH</p>
              <p className="text-xs text-gray-600 text-center mb-2">Saldo Fisik - Saldo Sistem</p>
              <p className={`text-4xl font-black text-center ${
                selisih === 0 ? 'text-green-700' : 
                selisih > 0 ? 'text-blue-700' : 
                'text-red-700'
              }`}>
                {selisih > 0 ? '+' : ''}{formatRupiah(selisih)}
              </p>
            </div>
          </div>
          
          {/* Status Message */}
          {selisih === 0 && (
            <div className="bg-green-500 text-white p-6 rounded-xl text-center shadow-lg mt-4">
              <p className="text-3xl font-black mb-2">✅ UANG SESUAI</p>
              <p className="text-lg font-semibold">TIDAK ADA SELISIH</p>
              <p className="text-sm mt-2 opacity-90">Kas fisik sudah cocok dengan data sistem</p>
            </div>
          )}
          
          {selisih > 0 && (
            <div className="bg-blue-500 text-white p-6 rounded-xl text-center shadow-lg mt-4">
              <p className="text-2xl font-black mb-2">💰 UANG LEBIH</p>
              <p className="text-lg font-semibold">Kelebihan: {formatRupiah(selisih)}</p>
              <p className="text-sm mt-2 opacity-90">
                Ada uang lebih atau transaksi belum diinput
              </p>
            </div>
          )}
          
          {selisih < 0 && (
            <div className="bg-red-500 text-white p-6 rounded-xl text-center shadow-lg mt-4">
              <p className="text-2xl font-black mb-2">⚠️ UANG KURANG</p>
              <p className="text-lg font-semibold">Kekurangan: {formatRupiah(Math.abs(selisih))}</p>
              <p className="text-sm mt-2 opacity-90">
                Periksa pengeluaran atau uang yang terpakai
              </p>
            </div>
          )}
        </div>
      </div>

      {/* Rumus Perhitungan */}
      <div className="bg-white rounded-2xl shadow-lg p-6 border">
        <h4 className="font-bold text-lg mb-3 text-gray-800">📐 Rumus Perhitungan:</h4>
        <div className="bg-gray-50 p-4 rounded-lg space-y-3">
          <div>
            <p className="font-semibold text-gray-700 mb-1">Saldo Fisik:</p>
            <p className="font-mono text-sm text-gray-700">
              Uang Cash + Uang di Bank + Uang Non Tunai (dari sistem)
            </p>
            <p className="font-mono text-sm text-emerald-600">
              {formatRupiah(totalUangCash)} + {formatRupiah(uangDiBank)} + {formatRupiah(uangNonTunaiData)} = {formatRupiah(saldoFisik)}
            </p>
          </div>
          
          <div className="border-t pt-3">
            <p className="font-semibold text-gray-700 mb-1">Saldo Sistem:</p>
            <p className="font-mono text-sm text-gray-700">
              Total Pemasukan - Total Pengeluaran
            </p>
            <p className="font-mono text-sm text-blue-600">
              {formatRupiah(totalPenerimaanData)} - {formatRupiah(totalPengeluaranData)} = {formatRupiah(saldoSistem)}
            </p>
          </div>
          
          <div className="border-t pt-3">
            <p className="font-semibold text-gray-700 mb-1">Selisih:</p>
            <p className="font-mono text-sm text-gray-700">
              Saldo Fisik - Saldo Sistem
            </p>
            <p className="font-mono text-sm text-purple-600">
              {formatRupiah(saldoFisik)} - {formatRupiah(saldoSistem)} = {formatRupiah(selisih)}
            </p>
          </div>
        </div>
      </div>

      {/* Panduan */}
      <div className="bg-gradient-to-r from-amber-50 to-orange-50 rounded-2xl p-6 border border-amber-200">
        <h4 className="font-bold text-amber-800 mb-3 flex items-center gap-2">
          <span className="text-2xl">💡</span>
          Cara Menggunakan:
        </h4>
        <ul className="space-y-2 text-sm text-gray-700">
          <li className="flex items-start gap-2">
            <span className="font-bold text-amber-600">1.</span>
            <span>Hitung uang cash fisik di kas, masukkan jumlah pcs per nominal</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-amber-600">2.</span>
            <span>Masukkan jumlah uang yang ada di rekening bank secara manual</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-amber-600">3.</span>
            <span>Sistem akan otomatis menghitung uang non tunai dari data penerimaan (Transfer + QRIS)</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-amber-600">4.</span>
            <span>Sistem akan otomatis menghitung pengeluaran dari data yang sudah diinput</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-amber-600">5.</span>
            <span>Sistem akan membandingkan Saldo Fisik dengan Saldo Sistem</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-emerald-600">✅</span>
            <span><strong>Selisih = 0</strong> = Kas fisik sesuai dengan saldo sistem</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-blue-600">💰</span>
            <span><strong>Selisih (+)</strong> = Kas fisik lebih banyak dari saldo sistem</span>
          </li>
          <li className="flex items-start gap-2">
            <span className="font-bold text-red-600">⚠️</span>
            <span><strong>Selisih (-)</strong> = Kas fisik kurang dari saldo sistem, periksa transaksi</span>
          </li>
        </ul>
        
        <div className="mt-4 bg-white p-4 rounded-lg border-2 border-amber-300">
          <p className="font-bold text-amber-800 mb-2">Catatan Penting:</p>
          <p className="text-sm text-gray-700">
            • Uang Cash: Hitung fisik di kas dan input per nominal<br/>
            • Uang di Bank: Input manual saldo rekening saat ini<br/>
            • Uang Non Tunai: Diambil otomatis dari data penerimaan (Transfer & QRIS)<br/>
            • Pengeluaran: Diambil otomatis dari data pengeluaran yang sudah dicatat<br/>
            • Pastikan semua transaksi sudah diinput untuk hasil yang akurat
          </p>
        </div>
      </div>
    </div>
  );
}

function LaporanView({ data, formatRupiah }) {
  const [copied, setCopied] = useState(false);

  const generateReport = () => {
    const now = new Date();
    const tanggal = now.toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' });
    const waktu = now.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
    
    const totalPenerimaan = data.penerimaan.reduce((sum, p) => sum + (p.jumlah || 0), 0);
    const totalZakatFitrah = data.penerimaan.filter(p => p.jenis === 'Zakat Fitrah').reduce((sum, p) => sum + (p.jumlah || 0), 0);
    const totalZakatMaal = data.penerimaan.filter(p => p.jenis === 'Zakat Maal').reduce((sum, p) => sum + (p.jumlah || 0), 0);
    const totalZakatProfesi = data.penerimaan.filter(p => p.jenis === 'Zakat Profesi').reduce((sum, p) => sum + (p.jumlah || 0), 0);
    const totalFidyah = data.penerimaan.filter(p => p.jenis === 'Fidyah').reduce((sum, p) => sum + (p.jumlah || 0), 0);
    const totalInfak = data.penerimaan.filter(p => p.jenis === 'Infak').reduce((sum, p) => sum + (p.jumlah || 0), 0);
    const totalBeras = data.penerimaan.reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);
    const berasFidyah = data.penerimaan.filter(p => p.jenisBeras === 'Fidyah').reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);
    const berasFitrah = data.penerimaan.filter(p => p.jenisBeras === 'Fitrah').reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);
    const berasSedekah = data.penerimaan.filter(p => p.jenisBeras === 'Sedekah').reduce((sum, p) => sum + (p.jumlahBeras || 0), 0);

    const targetZakat = data.settings.targetZakatFitrah || 0;
    const kurang = Math.max(targetZakat - totalZakatFitrah, 0);

    let report = `📢 *Update Terkini* Penerimaan Zakat, Fidyah, dan Sedekah
dari Counter Zakat ZIS *${data.settings.namaMasjid}*

📅 *Tanggal*: ${tanggal}
⏰ *Pukul*: ${waktu}

📌 *Total Muzaki yang Masuk*: ${data.penerimaan.length} orang
`;

    if (targetZakat > 0) {
      report += `
🎯 *Target Zakat Fitrah*: ${formatRupiah(targetZakat)}
✅ *Zakat Fitrah yang Masuk*: ${formatRupiah(totalZakatFitrah)}
⚠️ *Total Kebutuhan (Kurang)*: ${formatRupiah(kurang)}
`;
    }

    report += `
💰 *Uang*:
- Zakat Fitrah: ${formatRupiah(totalZakatFitrah)}
- Zakat Maal: ${formatRupiah(totalZakatMaal)}
- Zakat Profesi: ${formatRupiah(totalZakatProfesi)}
- Fidyah: ${formatRupiah(totalFidyah)}
- Sedekah: ${formatRupiah(totalInfak)}
- *Total*: ${formatRupiah(totalPenerimaan)}
`;

    if (totalBeras > 0) {
      report += `
🌾 *Beras*:
- Fidyah: ${berasFidyah.toFixed(2)} Kg
- Fitrah: ${berasFitrah.toFixed(2)} Kg
- Sedekah: ${berasSedekah.toFixed(2)} Kg
- *Total*: ${totalBeras.toFixed(2)} Kg
`;
    }

    if (data.settings.tanggalDistribusi) {
      const tglDist = new Date(data.settings.tanggalDistribusi);
      const tglDistStr = tglDist.toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' });
      report += `
📅 *Tanggal Distribusi*: ${tglDistStr}
`;
    }

    report += `
📍 *Jadwal Penerimaan Zakat*`;

    if (data.settings.jadwalMasjid) {
      report += `
🏠 Di Masjid: ${data.settings.jadwalMasjid}`;
    }

    const hasCluster = data.settings.jadwalSanur || data.settings.jadwalTampakSiring || data.settings.jadwalNusaDua;
    if (hasCluster) {
      report += `

🏡 Di Cluster:`;
      if (data.settings.jadwalSanur) {
        report += `
1️⃣ Sanur: ${data.settings.jadwalSanur}`;
      }
      if (data.settings.jadwalTampakSiring) {
        report += `
2️⃣ Tampak Siring: ${data.settings.jadwalTampakSiring}`;
      }
      if (data.settings.jadwalNusaDua) {
        report += `
3️⃣ Nusa Dua: ${data.settings.jadwalNusaDua}`;
      }
    }

    report += `

Distribusi zakat fitrah dan fidyah akan dilakukan kepada 
semua Mustahik terdaftar dan yayasan penerima zakat.

Syukron wa jazakumullahu khoiron kepada para Muzaki.
Semoga Allah membersihkan harta kita, memberkahi sisa yang ada, 
dan menjaga kesuciannya.

🌿 *Wassalamu'alaikum warahmatullahi wabarakatuh.*

🤲 *Panitia ZIS ${data.settings.namaMasjid}*`;

    return report;
  };

  const copyReport = () => {
    navigator.clipboard.writeText(generateReport());
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <div className="space-y-6">
      <h2 className="text-3xl font-bold">Laporan</h2>

      <div className="bg-white rounded-2xl shadow-lg p-6">
        <div className="flex justify-between items-center mb-4">
          <h3 className="text-xl font-bold">Laporan WhatsApp</h3>
          <button
            onClick={copyReport}
            className="flex items-center gap-2 bg-emerald-600 text-white px-6 py-3 rounded-xl hover:bg-emerald-700 font-semibold"
          >
            <Copy className="w-5 h-5" />
            {copied ? 'Tersalin!' : 'Salin Laporan'}
          </button>
        </div>
        <div className="bg-gray-50 p-6 rounded-xl border">
          <pre className="whitespace-pre-wrap font-sans text-sm">{generateReport()}</pre>
        </div>
      </div>
    </div>
  );
}

function UsersView() {
  const [users, setUsers] = useState([]);
  const [showModal, setShowModal] = useState(false);
  const [editData, setEditData] = useState(null);

  useEffect(() => {
    const loadUsers = async () => {
      const data = await loadData('masjid-users', []);
      setUsers(data);
    };
    loadUsers();
  }, []);

  const handleSave = async (userData) => {
    let updated;
    if (editData) {
      updated = users.map(u => u.id === editData.id ? { ...userData, id: editData.id } : u);
    } else {
      updated = [...users, { ...userData, id: Date.now() }];
    }
    setUsers(updated);
    await saveData('masjid-users', updated);
    setShowModal(false);
    setEditData(null);
  };

  const handleDelete = async (id) => {
    if (confirm('Hapus user ini?')) {
      const updated = users.filter(u => u.id !== id);
      setUsers(updated);
      await saveData('masjid-users', updated);
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h2 className="text-3xl font-bold">Kelola User</h2>
        <button
          onClick={() => { setEditData(null); setShowModal(true); }}
          className="flex items-center gap-2 bg-gradient-to-r from-indigo-600 to-purple-600 text-white px-6 py-3 rounded-xl hover:from-indigo-700 hover:to-purple-700 font-semibold shadow-lg"
        >
          <PlusCircle className="w-5 h-5" />
          Tambah User
        </button>
      </div>

      <div className="bg-white rounded-2xl shadow-lg overflow-hidden">
        <table className="w-full">
          <thead className="bg-gradient-to-r from-indigo-600 to-purple-600 text-white">
            <tr>
              <th className="px-6 py-4 text-left">Username</th>
              <th className="px-6 py-4 text-left">Nama</th>
              <th className="px-6 py-4 text-left">Role</th>
              <th className="px-6 py-4 text-center">Aksi</th>
            </tr>
          </thead>
          <tbody>
            {users.map(user => (
              <tr key={user.id} className="border-b hover:bg-indigo-50">
                <td className="px-6 py-4">{user.username}</td>
                <td className="px-6 py-4">{user.nama}</td>
                <td className="px-6 py-4">
                  <span className={`px-3 py-1 rounded-full text-sm ${
                    user.role === 'Admin' ? 'bg-purple-100 text-purple-700' : 'bg-blue-100 text-blue-700'
                  }`}>
                    {user.role}
                  </span>
                </td>
                <td className="px-6 py-4 text-center">
                  <div className="flex gap-2 justify-center">
                    <button onClick={() => { setEditData(user); setShowModal(true); }} className="text-blue-600 hover:text-blue-800 p-2 rounded-lg hover:bg-blue-50">
                      <Edit2 className="w-5 h-5" />
                    </button>
                    <button onClick={() => handleDelete(user.id)} className="text-red-600 hover:text-red-800 p-2 rounded-lg hover:bg-red-50" disabled={user.username === 'admin'}>
                      <X className="w-5 h-5" />
                    </button>
                  </div>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {showModal && (
        <UserModal
          data={editData}
          onSave={handleSave}
          onClose={() => { setShowModal(false); setEditData(null); }}
        />
      )}
    </div>
  );
}

function SettingsView({ settings, onSave }) {
  const [formData, setFormData] = useState(settings);
  const [newKategori, setNewKategori] = useState('');

  const handleSubmit = () => {
    onSave(formData);
    alert('Pengaturan berhasil disimpan!');
  };

  const handleAddKategori = () => {
    if (newKategori.trim()) {
      const updatedKategori = [...(formData.kategoriPengeluaran || []), newKategori.trim()];
      setFormData({...formData, kategoriPengeluaran: updatedKategori});
      setNewKategori('');
    }
  };

  const handleRemoveKategori = (index) => {
    const updatedKategori = formData.kategoriPengeluaran.filter((_, i) => i !== index);
    setFormData({...formData, kategoriPengeluaran: updatedKategori});
  };

  return (
    <div className="space-y-6 max-w-4xl">
      <h2 className="text-3xl font-bold">Pengaturan</h2>

      <div className="bg-white rounded-2xl shadow-lg p-8 space-y-6">
        {/* Informasi Masjid */}
        <div className="border-b pb-6">
          <h3 className="text-lg font-bold mb-4 text-emerald-700">Informasi Masjid</h3>
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold mb-2">Nama Masjid</label>
              <input
                type="text"
                value={formData.namaMasjid}
                onChange={(e) => setFormData({...formData, namaMasjid: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-emerald-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Target Zakat Fitrah (Rp)</label>
              <input
                type="number"
                value={formData.targetZakatFitrah === 0 ? '' : formData.targetZakatFitrah}
                onChange={(e) => {
                  const value = e.target.value;
                  setFormData({...formData, targetZakatFitrah: value === '' ? 0 : parseFloat(value) || 0});
                }}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-emerald-500"
                placeholder="Masukkan target dalam rupiah"
              />
              <p className="text-xs text-gray-500 mt-1">Kosongkan atau isi 0 jika tidak ingin menampilkan target</p>
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Tanggal Distribusi</label>
              <input
                type="date"
                value={formData.tanggalDistribusi}
                onChange={(e) => setFormData({...formData, tanggalDistribusi: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-emerald-500"
              />
            </div>
          </div>
        </div>

        {/* Informasi Rekening */}
        <div className="border-b pb-6">
          <h3 className="text-lg font-bold mb-4 text-blue-700">Informasi Rekening</h3>
          <div className="space-y-4">
            <div className="grid grid-cols-2 gap-4">
              <div>
                <label className="block text-sm font-semibold mb-2">Nama Bank</label>
                <input
                  type="text"
                  value={formData.namaBank || ''}
                  onChange={(e) => setFormData({...formData, namaBank: e.target.value})}
                  className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500"
                  placeholder="Bank Syariah Indonesia"
                />
              </div>
              <div>
                <label className="block text-sm font-semibold mb-2">Kode Bank</label>
                <input
                  type="text"
                  value={formData.kodeBank || ''}
                  onChange={(e) => setFormData({...formData, kodeBank: e.target.value})}
                  className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500"
                  placeholder="451"
                />
              </div>
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Nomor Rekening</label>
              <input
                type="text"
                value={formData.noRekening || ''}
                onChange={(e) => setFormData({...formData, noRekening: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500"
                placeholder="7019291698"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Atas Nama</label>
              <input
                type="text"
                value={formData.atasNama || ''}
                onChange={(e) => setFormData({...formData, atasNama: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500"
                placeholder="Eko Andri QQ BAITUL HIKMAH"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">No WhatsApp Konsultasi</label>
              <input
                type="text"
                value={formData.noWAKonsultasi || ''}
                onChange={(e) => setFormData({...formData, noWAKonsultasi: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500"
                placeholder="6289601705274"
              />
              <p className="text-xs text-gray-500 mt-1">Format: 628xxx (tanpa tanda +)</p>
            </div>
          </div>
        </div>

        {/* Kategori Pengeluaran */}
        <div className="border-b pb-6">
          <h3 className="text-lg font-bold mb-4 text-red-700">Kategori Pengeluaran</h3>
          <div className="space-y-3">
            <div className="flex gap-2">
              <input
                type="text"
                value={newKategori}
                onChange={(e) => setNewKategori(e.target.value)}
                onKeyPress={(e) => e.key === 'Enter' && handleAddKategori()}
                className="flex-1 px-4 py-3 border rounded-xl focus:ring-2 focus:ring-red-500"
                placeholder="Tambah kategori baru..."
              />
              <button
                type="button"
                onClick={handleAddKategori}
                className="px-6 py-3 bg-red-600 text-white rounded-xl hover:bg-red-700 font-semibold whitespace-nowrap"
              >
                + Tambah
              </button>
            </div>

            <div className="space-y-2">
              {(formData.kategoriPengeluaran || []).map((kat, index) => (
                <div key={index} className="flex items-center justify-between bg-red-50 p-3 rounded-lg">
                  <span className="text-gray-700">{kat}</span>
                  <button
                    type="button"
                    onClick={() => handleRemoveKategori(index)}
                    className="text-red-600 hover:text-red-800 hover:bg-red-100 p-1 rounded"
                  >
                    <X className="w-4 h-4" />
                  </button>
                </div>
              ))}
            </div>
            <p className="text-xs text-gray-500">Kategori ini akan muncul di dropdown saat input pengeluaran</p>
          </div>
        </div>

        {/* Jadwal Penerimaan */}
        <div>
          <h3 className="text-lg font-bold mb-4 text-purple-700">Jadwal Penerimaan Zakat</h3>
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold mb-2">Jadwal di Masjid</label>
              <input
                type="text"
                value={formData.jadwalMasjid}
                onChange={(e) => setFormData({...formData, jadwalMasjid: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                placeholder="Contoh: 23 Februari 2026"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Jadwal Cluster Sanur</label>
              <input
                type="text"
                value={formData.jadwalSanur}
                onChange={(e) => setFormData({...formData, jadwalSanur: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                placeholder="Contoh: 23 Februari 2026"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Jadwal Cluster Tampak Siring</label>
              <input
                type="text"
                value={formData.jadwalTampakSiring}
                onChange={(e) => setFormData({...formData, jadwalTampakSiring: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                placeholder="Contoh: 23 Februari 2026"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Jadwal Cluster Nusa Dua</label>
              <input
                type="text"
                value={formData.jadwalNusaDua}
                onChange={(e) => setFormData({...formData, jadwalNusaDua: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                placeholder="Contoh: 23 Februari 2026"
              />
            </div>
          </div>
        </div>

        <button
          onClick={handleSubmit}
          className="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-4 rounded-xl hover:from-emerald-700 hover:to-teal-700 font-semibold text-lg shadow-lg"
        >
          Simpan Pengaturan
        </button>
      </div>
    </div>
  );
}

// Modals akan ditambahkan di pesan berikutnya karena sudah mencapai limit

function PenerimaanModal({ data, onSave, onClose }) {
  const [formData, setFormData] = useState(data || {
    tanggal: new Date().toISOString().split('T')[0],
    jenis: 'Zakat Fitrah',
    donatur: '',
    alamat: '',
    noHP: '',
    jumlahKeluarga: '',
    anggotaKeluarga: [],
    lokasi: 'Masjid',
    jumlah: 0,
    jumlahBeras: 0,
    jenisBeras: '',
    keterangan: '',
    metodePembayaran: 'Tunai'
  });
  const [showAnggotaForm, setShowAnggotaForm] = useState(false);
  const [namaAnggota, setNamaAnggota] = useState('');

  const handleAddAnggota = () => {
    if (namaAnggota.trim()) {
      setFormData({
        ...formData,
        anggotaKeluarga: [...formData.anggotaKeluarga, namaAnggota.trim()]
      });
      setNamaAnggota('');
      setShowAnggotaForm(false);
    }
  };

  const handleRemoveAnggota = (index) => {
    setFormData({
      ...formData,
      anggotaKeluarga: formData.anggotaKeluarga.filter((_, i) => i !== index)
    });
  };

  const handleSubmit = () => {
    if (!formData.donatur) {
      alert('Nama donatur harus diisi!');
      return;
    }
    if (!formData.jumlah && !formData.jumlahBeras) {
      alert('Isi minimal jumlah uang atau jumlah beras!');
      return;
    }
    if (formData.jumlahBeras > 0 && !formData.jenisBeras) {
      alert('Pilih jenis beras!');
      return;
    }
    onSave(formData);
  };

  const sendWhatsApp = () => {
    if (!formData.noHP) {
      alert('Nomor HP harus diisi untuk mengirim WhatsApp!');
      return;
    }

    const formatRupiah = (amount) => {
      return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0
      }).format(amount);
    };

    let message = `📢 *Konfirmasi Penerimaan Zakat*

👤 *Nama Kepala Keluarga:* ${formData.donatur}`;

    if (formData.alamat) {
      message += `
📍 *Alamat:* ${formData.alamat}`;
    }

    message += `
📱 *No HP:* ${formData.noHP}`;

    if (formData.jumlahKeluarga || formData.anggotaKeluarga.length > 0) {
      message += `
👥 *Jumlah Keluarga:* ${formData.jumlahKeluarga || formData.anggotaKeluarga.length} orang`;
    }

    if (formData.anggotaKeluarga.length > 0) {
      message += `

👨‍👩‍👧‍👦 *Anggota Keluarga:*
${formData.anggotaKeluarga.map((nama, i) => `${i + 1}. ${nama}`).join('\n')}`;
    }

    message += `

💵 *Metode Pembayaran:* ${formData.metodePembayaran}
📍 *Lokasi Counter:* ${formData.lokasi}

💰 *${formData.jenis}:* ${formatRupiah(formData.jumlah || 0)}`;

    if (formData.jumlahBeras > 0) {
      message += ` + ${formData.jumlahBeras} Kg Beras (${formData.jenisBeras})`;
    }

    message += `

🙏 *Terima kasih atas zakat & sedekah yang telah Anda tunaikan.*
Semoga Allah menerima amal ibadah kita. Aamiin 🙏🏻

🏦 *Transfer Zakat/Sedekah:*
• ${data?.settings?.namaBank || 'Bank Syariah Indonesia'} (${data?.settings?.kodeBank || '451'})
• ${data?.settings?.noRekening || '7019291698'}
• a/n ${data?.settings?.atasNama || 'Eko Andri QQ BAITUL HIKMAH'}

📞 *Konsultasi:* wa.me/${data?.settings?.noWAKonsultasi || '6289601705274'}

Jazakumullahu khairan 🙏`;

    const phoneNumber = formData.noHP.replace(/\D/g, '');
    const whatsappURL = `https://wa.me/${phoneNumber}?text=${encodeURIComponent(message)}`;
    window.open(whatsappURL, '_blank');
  };

  return (
    <div className="fixed inset-0 bg-black/50 z-50 overflow-y-auto">
      <div className="min-h-screen flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-2xl max-w-4xl w-full p-8 my-8">
          <div className="flex justify-between items-center mb-6">
            <h3 className="text-2xl font-bold">{data ? 'Edit' : 'Tambah'} Penerimaan</h3>
            <button onClick={onClose} className="text-gray-400 hover:text-gray-600">
              <X className="w-6 h-6" />
            </button>
          </div>

          <div className="space-y-6 max-h-[60vh] overflow-y-auto pr-2">
            {/* Informasi Kepala Keluarga */}
            <div className="bg-emerald-50 p-4 rounded-xl">
              <h4 className="font-bold text-emerald-800 mb-3 flex items-center gap-2">
                <User className="w-5 h-5" />
                Informasi Kepala Keluarga
              </h4>
              <div className="space-y-3">
                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <label className="block text-sm font-semibold mb-2">Nama Kepala Keluarga *</label>
                    <input
                      type="text"
                      value={formData.donatur}
                      onChange={(e) => setFormData({...formData, donatur: e.target.value})}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-emerald-500"
                      placeholder="Nama lengkap"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-semibold mb-2">No HP/WhatsApp</label>
                    <input
                      type="text"
                      value={formData.noHP}
                      onChange={(e) => {
                        let value = e.target.value.replace(/\D/g, '');
                        if (value.startsWith('0') && value.length > 1) {
                          value = '62' + value.substring(1);
                        }
                        setFormData({...formData, noHP: value});
                      }}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-emerald-500"
                      placeholder="08xxx (otomatis jadi 628xxx)"
                    />
                    <p className="text-xs text-gray-500 mt-1">Ketik 08xxx akan otomatis jadi 628xxx</p>
                  </div>
                </div>
                <div>
                  <label className="block text-sm font-semibold mb-2">Alamat Lengkap</label>
                  <textarea
                    value={formData.alamat}
                    onChange={(e) => setFormData({...formData, alamat: e.target.value})}
                    className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-emerald-500"
                    rows="2"
                    placeholder="Alamat lengkap (Blok/No, RT/RW, dll)"
                  />
                </div>
              </div>
            </div>

            {/* Anggota Keluarga */}
            <div className="bg-blue-50 p-4 rounded-xl">
              <div className="flex justify-between items-center mb-3">
                <h4 className="font-bold text-blue-800 flex items-center gap-2">
                  <Users className="w-5 h-5" />
                  Anggota Keluarga
                </h4>
                <button
                  type="button"
                  onClick={() => setShowAnggotaForm(true)}
                  className="flex items-center gap-2 bg-blue-600 text-white px-3 py-2 rounded-lg hover:bg-blue-700 text-sm"
                >
                  <PlusCircle className="w-4 h-4" />
                  Tambah Anggota
                </button>
              </div>

              <div className="mb-3">
                <label className="block text-sm font-semibold mb-2">Jumlah Keluarga</label>
                <input
                  type="number"
                  value={formData.jumlahKeluarga === 0 ? '' : formData.jumlahKeluarga}
                  onChange={(e) => {
                    const value = e.target.value;
                    setFormData({...formData, jumlahKeluarga: value === '' ? '' : value});
                  }}
                  className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500"
                  placeholder="Total anggota keluarga"
                />
              </div>

              {showAnggotaForm && (
                <div className="bg-white p-3 rounded-lg border-2 border-blue-300 mb-3">
                  <label className="block text-sm font-semibold mb-2">Nama Anggota Keluarga</label>
                  <input
                    type="text"
                    value={namaAnggota}
                    onChange={(e) => setNamaAnggota(e.target.value)}
                    onKeyPress={(e) => e.key === 'Enter' && handleAddAnggota()}
                    className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 mb-2"
                    placeholder="Nama lengkap anggota"
                    autoFocus
                  />
                  <div className="flex gap-2">
                    <button
                      type="button"
                      onClick={handleAddAnggota}
                      className="flex-1 bg-blue-600 text-white px-3 py-2 rounded-lg hover:bg-blue-700 text-sm"
                    >
                      Simpan
                    </button>
                    <button
                      type="button"
                      onClick={() => { setShowAnggotaForm(false); setNamaAnggota(''); }}
                      className="flex-1 bg-gray-200 text-gray-700 px-3 py-2 rounded-lg hover:bg-gray-300 text-sm"
                    >
                      Batal
                    </button>
                  </div>
                </div>
              )}

              {formData.anggotaKeluarga.length > 0 && (
                <div className="space-y-2">
                  {formData.anggotaKeluarga.map((nama, index) => (
                    <div key={index} className="flex items-center justify-between bg-white p-3 rounded-lg border">
                      <span className="text-gray-700">{index + 1}. {nama}</span>
                      <button
                        type="button"
                        onClick={() => handleRemoveAnggota(index)}
                        className="text-red-600 hover:text-red-800 hover:bg-red-50 p-1 rounded"
                      >
                        <X className="w-4 h-4" />
                      </button>
                    </div>
                  ))}
                </div>
              )}
            </div>

            {/* Detail Transaksi */}
            <div className="bg-purple-50 p-4 rounded-xl">
              <h4 className="font-bold text-purple-800 mb-3 flex items-center gap-2">
                <DollarSign className="w-5 h-5" />
                Detail Transaksi
              </h4>
              <div className="space-y-3">
                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <label className="block text-sm font-semibold mb-2">Tanggal</label>
                    <input
                      type="date"
                      value={formData.tanggal}
                      onChange={(e) => setFormData({...formData, tanggal: e.target.value})}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-semibold mb-2">Jenis</label>
                    <select
                      value={formData.jenis}
                      onChange={(e) => setFormData({...formData, jenis: e.target.value})}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                    >
                      <option>Zakat Fitrah</option>
                      <option>Zakat Maal</option>
                      <option>Zakat Profesi</option>
                      <option>Infak</option>
                      <option>Fidyah</option>
                    </select>
                  </div>
                </div>

                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <label className="block text-sm font-semibold mb-2">Metode Pembayaran</label>
                    <select
                      value={formData.metodePembayaran}
                      onChange={(e) => setFormData({...formData, metodePembayaran: e.target.value})}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                    >
                      <option>Tunai</option>
                      <option>Transfer</option>
                      <option>QRIS</option>
                    </select>
                  </div>
                  <div>
                    <label className="block text-sm font-semibold mb-2">Lokasi Counter</label>
                    <select
                      value={formData.lokasi}
                      onChange={(e) => setFormData({...formData, lokasi: e.target.value})}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                    >
                      <option>Masjid</option>
                      <option>Cluster Sanur</option>
                      <option>Cluster Tampak Siring</option>
                      <option>Cluster Nusa Dua</option>
                    </select>
                  </div>
                </div>

                <div className="grid grid-cols-2 gap-3">
                  <div>
                    <label className="block text-sm font-semibold mb-2">Jumlah Uang (Rp)</label>
                    <input
                      type="number"
                      value={formData.jumlah === 0 ? '' : formData.jumlah}
                      onChange={(e) => {
                        const value = e.target.value;
                        setFormData({...formData, jumlah: value === '' ? 0 : parseFloat(value) || 0});
                      }}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                      placeholder="0"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-semibold mb-2">Jumlah Beras (Kg)</label>
                    <input
                      type="number"
                      step="0.01"
                      value={formData.jumlahBeras === 0 ? '' : formData.jumlahBeras}
                      onChange={(e) => {
                        const value = e.target.value;
                        setFormData({...formData, jumlahBeras: value === '' ? 0 : parseFloat(value) || 0});
                      }}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                      placeholder="0"
                    />
                  </div>
                </div>

                {formData.jumlahBeras > 0 && (
                  <div>
                    <label className="block text-sm font-semibold mb-2">Jenis Beras</label>
                    <select
                      value={formData.jenisBeras}
                      onChange={(e) => setFormData({...formData, jenisBeras: e.target.value})}
                      className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                    >
                      <option value="">Pilih jenis beras</option>
                      <option>Fidyah</option>
                      <option>Fitrah</option>
                      <option>Sedekah</option>
                    </select>
                  </div>
                )}

                <div>
                  <label className="block text-sm font-semibold mb-2">Keterangan</label>
                  <textarea
                    value={formData.keterangan}
                    onChange={(e) => setFormData({...formData, keterangan: e.target.value})}
                    className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                    rows="2"
                    placeholder="Keterangan tambahan (opsional)"
                  />
                </div>
              </div>
            </div>
          </div>

          <div className="flex gap-3 mt-6">
            <button
              onClick={onClose}
              className="flex-1 px-6 py-3 border-2 text-gray-700 rounded-xl hover:bg-gray-50 font-semibold"
            >
              Batal
            </button>
            <button
              onClick={handleSubmit}
              className="flex-1 px-6 py-3 bg-gradient-to-r from-emerald-600 to-teal-600 text-white rounded-xl hover:from-emerald-700 hover:to-teal-700 font-semibold shadow-lg"
            >
              {data ? 'Update' : 'Simpan'}
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}

function PengeluaranModal({ data, mustahik, kategoriList, onSave, onClose }) {
  const [formData, setFormData] = useState(data || {
    tanggal: new Date().toISOString().split('T')[0],
    penerima: '',
    kategori: kategoriList[0] || 'Distribusi Zakat',
    jumlah: 0,
    keterangan: ''
  });

  const handleSubmit = () => {
    if (!formData.penerima || !formData.jumlah) {
      alert('Penerima dan jumlah harus diisi!');
      return;
    }
    onSave(formData);
  };

  return (
    <div className="fixed inset-0 bg-black/50 z-50 overflow-y-auto">
      <div className="min-h-screen flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-2xl max-w-md w-full p-8">
          <div className="flex justify-between items-center mb-6">
            <h3 className="text-2xl font-bold">{data ? 'Edit' : 'Tambah'} Pengeluaran</h3>
            <button onClick={onClose} className="text-gray-400 hover:text-gray-600">
              <X className="w-6 h-6" />
            </button>
          </div>

          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold mb-2">Tanggal</label>
              <input
                type="date"
                value={formData.tanggal}
                onChange={(e) => setFormData({...formData, tanggal: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-red-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Penerima</label>
              <select
                value={formData.penerima}
                onChange={(e) => setFormData({...formData, penerima: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-red-500"
              >
                <option value="">Pilih penerima</option>
                {mustahik.map(m => (
                  <option key={m.id} value={m.nama}>{m.nama}</option>
                ))}
                <option value="Lainnya">Lainnya</option>
              </select>
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Kategori</label>
              <select
                value={formData.kategori}
                onChange={(e) => setFormData({...formData, kategori: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-red-500"
              >
                {kategoriList.map((kat, index) => (
                  <option key={index} value={kat}>{kat}</option>
                ))}
              </select>
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Jumlah (Rp)</label>
              <input
                type="number"
                value={formData.jumlah === 0 ? '' : formData.jumlah}
                onChange={(e) => {
                  const value = e.target.value;
                  setFormData({...formData, jumlah: value === '' ? 0 : parseFloat(value) || 0});
                }}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-red-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Keterangan</label>
              <textarea
                value={formData.keterangan}
                onChange={(e) => setFormData({...formData, keterangan: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-red-500"
                rows="3"
              />
            </div>
          </div>

          <div className="flex gap-3 mt-6">
            <button
              onClick={onClose}
              className="flex-1 px-6 py-3 border-2 text-gray-700 rounded-xl hover:bg-gray-50 font-semibold"
            >
              Batal
            </button>
            <button
              onClick={handleSubmit}
              className="flex-1 px-6 py-3 bg-gradient-to-r from-red-600 to-rose-600 text-white rounded-xl hover:from-red-700 hover:to-rose-700 font-semibold shadow-lg"
            >
              {data ? 'Update' : 'Simpan'}
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}

function MustahikModal({ data, onSave, onClose }) {
  const [formData, setFormData] = useState(data || {
    nama: '',
    alamat: '',
    noTelp: '',
    kategori: 'Fakir',
    keterangan: ''
  });

  const handleSubmit = () => {
    if (!formData.nama || !formData.alamat || !formData.noTelp) {
      alert('Nama, alamat, dan no telepon harus diisi!');
      return;
    }
    onSave(formData);
  };

  return (
    <div className="fixed inset-0 bg-black/50 z-50 overflow-y-auto">
      <div className="min-h-screen flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-2xl max-w-md w-full p-8">
          <div className="flex justify-between items-center mb-6">
            <h3 className="text-2xl font-bold">{data ? 'Edit' : 'Tambah'} Mustahik</h3>
            <button onClick={onClose} className="text-gray-400 hover:text-gray-600">
              <X className="w-6 h-6" />
            </button>
          </div>

          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold mb-2">Nama Lengkap</label>
              <input
                type="text"
                value={formData.nama}
                onChange={(e) => setFormData({...formData, nama: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Alamat</label>
              <textarea
                value={formData.alamat}
                onChange={(e) => setFormData({...formData, alamat: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                rows="2"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">No. Telepon</label>
              <input
                type="tel"
                value={formData.noTelp}
                onChange={(e) => setFormData({...formData, noTelp: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Kategori</label>
              <select
                value={formData.kategori}
                onChange={(e) => setFormData({...formData, kategori: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
              >
                <option>Fakir</option>
                <option>Miskin</option>
                <option>Amil</option>
                <option>Mualaf</option>
                <option>Riqab</option>
                <option>Gharimin</option>
                <option>Fisabilillah</option>
                <option>Ibnus Sabil</option>
              </select>
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Keterangan</label>
              <textarea
                value={formData.keterangan}
                onChange={(e) => setFormData({...formData, keterangan: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-purple-500"
                rows="2"
              />
            </div>
          </div>

          <div className="flex gap-3 mt-6">
            <button
              onClick={onClose}
              className="flex-1 px-6 py-3 border-2 text-gray-700 rounded-xl hover:bg-gray-50 font-semibold"
            >
              Batal
            </button>
            <button
              onClick={handleSubmit}
              className="flex-1 px-6 py-3 bg-gradient-to-r from-purple-600 to-indigo-600 text-white rounded-xl hover:from-purple-700 hover:to-indigo-700 font-semibold shadow-lg"
            >
              {data ? 'Update' : 'Simpan'}
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}

function UserModal({ data, onSave, onClose }) {
  const [formData, setFormData] = useState(data || {
    username: '',
    password: '',
    nama: '',
    role: 'Petugas'
  });

  const handleSubmit = () => {
    if (!formData.username || !formData.nama) {
      alert('Username dan nama harus diisi!');
      return;
    }
    if (!data && !formData.password) {
      alert('Password harus diisi!');
      return;
    }
    onSave(formData);
  };

  return (
    <div className="fixed inset-0 bg-black/50 z-50 overflow-y-auto">
      <div className="min-h-screen flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-2xl max-w-md w-full p-8">
          <div className="flex justify-between items-center mb-6">
            <h3 className="text-2xl font-bold">{data ? 'Edit' : 'Tambah'} User</h3>
            <button onClick={onClose} className="text-gray-400 hover:text-gray-600">
              <X className="w-6 h-6" />
            </button>
          </div>

          <div className="space-y-4">
            <div>
              <label className="block text-sm font-semibold mb-2">Username</label>
              <input
                type="text"
                value={formData.username}
                onChange={(e) => setFormData({...formData, username: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-indigo-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">
                Password {data && '(Kosongkan jika tidak diubah)'}
              </label>
              <input
                type="password"
                value={formData.password}
                onChange={(e) => setFormData({...formData, password: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-indigo-500"
                placeholder={data ? "Kosongkan jika tidak diubah" : ""}
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Nama Lengkap</label>
              <input
                type="text"
                value={formData.nama}
                onChange={(e) => setFormData({...formData, nama: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-indigo-500"
              />
            </div>

            <div>
              <label className="block text-sm font-semibold mb-2">Role</label>
              <select
                value={formData.role}
                onChange={(e) => setFormData({...formData, role: e.target.value})}
                className="w-full px-4 py-3 border rounded-xl focus:ring-2 focus:ring-indigo-500"
              >
                <option>Petugas</option>
                <option>Admin</option>
              </select>
            </div>
          </div>

          <div className="flex gap-3 mt-6">
            <button
              onClick={onClose}
              className="flex-1 px-6 py-3 border-2 text-gray-700 rounded-xl hover:bg-gray-50 font-semibold"
            >
              Batal
            </button>
            <button
              onClick={handleSubmit}
              className="flex-1 px-6 py-3 bg-gradient-to-r from-indigo-600 to-purple-600 text-white rounded-xl hover:from-indigo-700 hover:to-purple-700 font-semibold shadow-lg"
            >
              {data ? 'Update' : 'Simpan'}
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
