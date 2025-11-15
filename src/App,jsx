import React, { useState, useEffect } from 'react';
import { Camera, Leaf, Droplets, Sun, AlertCircle, CheckCircle, ChevronRight, TrendingUp, Plus, X, Crown, LogOut, Bell } from 'lucide-react';

// Simple in-memory storage (will be replaced with real database later)
const storage = {
  data: {},
  async get(key) {
    return this.data[key] ? { value: this.data[key] } : null;
  },
  async set(key, value) {
    this.data[key] = value;
    return true;
  },
  async delete(key) {
    delete this.data[key];
    return true;
  },
  async list(prefix) {
    const keys = Object.keys(this.data).filter(k => k.startsWith(prefix));
    return { keys };
  }
};

export default function App() {
  const [screen, setScreen] = useState('auth');
  const [authMode, setAuthMode] = useState('login');
  const [user, setUser] = useState(null);
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [authMessage, setAuthMessage] = useState('');
  const [authLoading, setAuthLoading] = useState(false);
  
  const [imageData, setImageData] = useState(null);
  const [plantName, setPlantName] = useState('');
  const [questionnaire, setQuestionnaire] = useState({ 
    lastWatered: '', 
    recentChanges: '', 
    soilCondition: '', 
    lightCondition: '', 
    symptoms: [] 
  });
  const [diagnosis, setDiagnosis] = useState(null);
  const [loading, setLoading] = useState(false);
  const [savedPlants, setSavedPlants] = useState([]);
  const [selectedPlant, setSelectedPlant] = useState(null);
  const [storageLoading, setStorageLoading] = useState(false);
  
  const [isPremium, setIsPremium] = useState(false);
  const [usageStats, setUsageStats] = useState({ diagnosesThisMonth: 0 });
  const [showUpgradeModal, setShowUpgradeModal] = useState(false);

  useEffect(() => { 
    checkAuth(); 
  }, []);

  const checkAuth = async () => {
    try {
      const authData = await storage.get('current-user');
      if (authData?.value) {
        const userData = JSON.parse(authData.value);
        setUser(userData);
        setIsPremium(userData.isPremium || false);
        setScreen('home');
        loadUserData(userData.email);
      }
    } catch (error) {
      console.log('No user');
    }
  };

  const loadUserData = async (userEmail) => {
    setStorageLoading(true);
    try {
      const result = await storage.list(`plant:${userEmail}:`);
      if (result?.keys) {
        const plantPromises = result.keys.map(async (key) => {
          const plantData = await storage.get(key);
          return plantData?.value ? JSON.parse(plantData.value) : null;
        });
        const plants = (await Promise.all(plantPromises)).filter(p => p);
        setSavedPlants(plants.sort((a, b) => b.id - a.id));
      }
      
      const stats = await storage.get(`usage:${userEmail}`);
      if (stats?.value) {
        const data = JSON.parse(stats.value);
        const now = new Date();
        const lastReset = new Date(data.lastReset);
        if (now.getMonth() !== lastReset.getMonth() || now.getFullYear() !== lastReset.getFullYear()) {
          setUsageStats({ diagnosesThisMonth: 0 });
        } else {
          setUsageStats({ diagnosesThisMonth: data.diagnosesThisMonth });
        }
      }
    } catch (error) {
      console.log('Load error:', error);
    } finally {
      setStorageLoading(false);
    }
  };

  const handleAuth = async () => {
    setAuthMessage('');
    setAuthLoading(true);
    
    if (!email || !password || password.length < 4) {
      setAuthMessage('Enter email and password (4+ chars)');
      setAuthLoading(false);
      return;
    }

    try {
      if (authMode === 'signup') {
        try {
          const exists = await storage.get(`user:${email}`);
          if (exists?.value) {
            setAuthMessage('User exists. Please login.');
            setAuthMode('login');
            setAuthLoading(false);
            return;
          }
        } catch (error) {}
        
        const userData = { email, password, isPremium: false, createdAt: new Date().toISOString() };
        await storage.set(`user:${email}`, JSON.stringify(userData));
        await storage.set('current-user', JSON.stringify(userData));
        setUser(userData);
        setIsPremium(false);
        setTimeout(() => { setScreen('home'); setAuthLoading(false); }, 300);
      } else {
        try {
          const result = await storage.get(`user:${email}`);
          if (!result?.value) {
            setAuthMessage('User not found. Sign up first.');
            setAuthMode('signup');
            setAuthLoading(false);
            return;
          }
          const userData = JSON.parse(result.value);
          if (userData.password !== password) {
            setAuthMessage('Incorrect password');
            setAuthLoading(false);
            return;
          }
          await storage.set('current-user', JSON.stringify(userData));
          setUser(userData);
          setIsPremium(userData.isPremium || false);
          setTimeout(() => { setScreen('home'); loadUserData(email); setAuthLoading(false); }, 300);
        } catch (error) {
          setAuthMessage('Login failed');
          setAuthLoading(false);
        }
      }
    } catch (error) {
      setAuthMessage('Auth error');
      setAuthLoading(false);
    }
  };

  const handleLogout = async () => {
    await storage.delete('current-user');
    setUser(null);
    setSavedPlants([]);
    setScreen('auth');
  };

  const savePlantToStorage = async (plant) => {
    for (let i = 0; i < 3; i++) {
      try {
        await storage.set(`plant:${user.email}:${plant.id}`, JSON.stringify(plant));
        return true;
      } catch (error) {
        if (i < 2) await new Promise(r => setTimeout(r, 100 * Math.pow(2, i)));
      }
    }
    return false;
  };

  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      if (!isPremium && savedPlants.length >= 3) {
        setShowUpgradeModal(true);
        return;
      }
      const reader = new FileReader();
      reader.onloadend = () => {
        setImageData(reader.result);
        setScreen('questionnaire');
      };
      reader.readAsDataURL(file);
    }
  };

  const symptoms = ['Yellow leaves', 'Brown spots', 'Wilting', 'Drooping', 'Leaf drop', 'Brown tips', 'Curling leaves', 'White residue', 'Holes', 'Sticky', 'Slow growth', 'Roots'];

  const toggleSymptom = (s) => {
    setQuestionnaire(p => ({
      ...p,
      symptoms: p.symptoms.includes(s) ? p.symptoms.filter(x => x !== s) : [...p.symptoms, s]
    }));
  };

  const calcWater = (light, soil) => {
    let d = 7;
    if (light === 'Direct sunlight') d = 3;
    else if (light === 'Bright indirect') d = 5;
    else if (light === 'Low light') d = 10;
    if (soil === 'Very wet' || soil === 'Soggy/waterlogged') d += 3;
    return new Date(Date.now() + d * 86400000);
  };

  const incUsage = async () => {
    const stats = { diagnosesThisMonth: usageStats.diagnosesThisMonth + 1 };
    setUsageStats(stats);
    await storage.set(`usage:${user.email}`, JSON.stringify({ ...stats, lastReset: new Date().toISOString() }));
  };

  const checkLimit = () => isPremium || usageStats.diagnosesThisMonth < 2;

  const getDiagnosis = async () => {
    if (!checkLimit()) {
      setShowUpgradeModal(true);
      setScreen('home');
      return;
    }

    setLoading(true);
    setScreen('diagnosis');

    try {
      const resp = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: { 
          'Content-Type': 'application/json',
          'x-api-key': import.meta.env.VITE_ANTHROPIC_API_KEY,
          'anthropic-version': '2023-06-01'
        },
        body: JSON.stringify({
          model: 'claude-sonnet-4-20250514',
          max_tokens: 1000,
          messages: [{ 
            role: 'user', 
            content: [
              { 
                type: 'image', 
                source: { 
                  type: 'base64', 
                  media_type: 'image/jpeg', 
                  data: imageData.split(',')[1] 
                }
              },
              { 
                type: 'text', 
                text: `Plant expert. Analyze.\nPlant: ${plantName}\nWatered: ${questionnaire.lastWatered}\nSoil: ${questionnaire.soilCondition}\nLight: ${questionnaire.lightCondition}\nSymptoms: ${questionnaire.symptoms.join(', ')}\n\nJSON only:\n{"primaryDiagnosis":"x","confidence":"high","explanation":"y","causes":["a"],"treatment":["b"],"timeline":"c","prevention":["d"]}` 
              }
            ]
          }]
        })
      });
      
      if (!resp.ok) {
        throw new Error(`API error: ${resp.status}`);
      }
      
      const data = await resp.json();
      const txt = data.content?.find(b => b.type === 'text')?.text || '';
      setDiagnosis(JSON.parse(txt.replace(/```json\n?|\n?```/g, '').trim()));
      await incUsage();
    } catch (error) {
      console.error('Diagnosis error:', error);
      setDiagnosis({ 
        primaryDiagnosis: 'Error', 
        confidence: 'low', 
        explanation: error.message, 
        causes: ['API Issue'], 
        treatment: ['Please try again'], 
        timeline: 'N/A', 
        prevention: [] 
      });
    } finally {
      setLoading(false);
    }
  };

  const savePlant = async () => {
    const plant = {
      id: Date.now(),
      name: plantName || 'My Plant',
      progressPhotos: [{ image: imageData, date: new Date().toISOString(), notes: 'Initial' }],
      diagnosis, 
      dateAdded: new Date().toISOString(), 
      questionnaire,
      wateringSchedule: {
        nextWatering: calcWater(questionnaire.lightCondition, questionnaire.soilCondition).toISOString(),
        lastWatered: new Date().toISOString()
      }
    };
    if (await savePlantToStorage(plant)) {
      setSavedPlants(p => [plant, ...p]);
      setScreen('home');
      setImageData(null); 
      setPlantName(''); 
      setQuestionnaire({ lastWatered: '', recentChanges: '', soilCondition: '', lightCondition: '', symptoms: [] }); 
      setDiagnosis(null);
    }
  };

  const markWatered = async (id) => {
    const plant = savedPlants.find(p => p.id === id);
    if (!plant) return;
    const updated = { 
      ...plant, 
      wateringSchedule: { 
        lastWatered: new Date().toISOString(), 
        nextWatering: calcWater(plant.questionnaire.lightCondition, 'Slightly moist').toISOString() 
      }
    };
    setSavedPlants(p => p.map(x => x.id === id ? updated : x));
    if (selectedPlant?.id === id) setSelectedPlant(updated);
    await savePlantToStorage(updated);
  };

  const addPhoto = async (id, photoData, notes) => {
    const plant = savedPlants.find(p => p.id === id);
    if (!plant) return;
    if (!isPremium && plant.progressPhotos.length >= 5) {
      setShowUpgradeModal(true);
      return;
    }
    const updated = { 
      ...plant, 
      progressPhotos: [...plant.progressPhotos, { image: photoData, date: new Date().toISOString(), notes: notes || 'Update' }]
    };
    if (await savePlantToStorage(updated)) {
      setSavedPlants(p => p.map(x => x.id === id ? updated : x));
      setSelectedPlant(updated);
    }
  };

  const getOverdue = () => savedPlants.filter(p => Math.ceil((new Date(p.wateringSchedule.nextWatering) - new Date()) / 86400000) < 0);

  const getWaterStatus = (next) => {
    const d = Math.ceil((new Date(next) - new Date()) / 86400000);
    if (d < 0) return { text: `Overdue ${Math.abs(d)}d`, color: 'text-red-600', bg: 'bg-red-50' };
    if (d === 0) return { text: 'Water today', color: 'text-orange-600', bg: 'bg-orange-50' };
    if (d === 1) return { text: 'Tomorrow', color: 'text-yellow-600', bg: 'bg-yellow-50' };
    return { text: `In ${d}d`, color: 'text-green-600', bg: 'bg-green-50' };
  };

  const delPlant = async (id) => {
    await storage.delete(`plant:${user.email}:${id}`);
    setSavedPlants(p => p.filter(x => x.id !== id));
    if (selectedPlant?.id === id) { setSelectedPlant(null); setScreen('home'); }
  };

  if (screen === 'auth') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-400 to-teal-600 flex items-center justify-center p-6">
        <div className="bg-white rounded-3xl shadow-2xl p-8 max-w-md w-full">
          <Leaf className="w-16 h-16 text-green-600 mx-auto mb-4" />
          <h1 className="text-3xl font-bold text-center mb-2">Plant Health</h1>
          <p className="text-center text-gray-600 mb-8">AI plant care</p>

          {authMessage && (
            <div className={`p-3 rounded-lg text-sm text-center mb-4 ${authMessage.includes('Success') ? 'bg-green-50 text-green-700' : 'bg-red-50 text-red-700'}`}>
              {authMessage}
            </div>
          )}
          
          <input 
            type="email" 
            value={email} 
            onChange={(e) => setEmail(e.target.value)} 
            placeholder="Email" 
            className="w-full px-4 py-3 border rounded-lg mb-4" 
          />
          <input 
            type="password" 
            value={password} 
            onChange={(e) => setPassword(e.target.value)} 
            placeholder="Password" 
            className="w-full px-4 py-3 border rounded-lg mb-4" 
          />

          <button 
            onClick={handleAuth} 
            disabled={authLoading} 
            className="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-3 rounded-lg mb-4"
          >
            {authLoading ? 'Processing...' : (authMode === 'login' ? 'Login' : 'Sign Up')}
          </button>

          <button 
            onClick={() => setAuthMode(authMode === 'login' ? 'signup' : 'login')} 
            className="w-full text-green-600 text-sm"
          >
            {authMode === 'login' ? 'Need account? Sign up' : 'Have account? Login'}
          </button>
        </div>
      </div>
    );
  }

  if (showUpgradeModal) {
    return (
      <div className="min-h-screen bg-purple-50 p-6">
        <div className="max-w-3xl mx-auto">
          <button onClick={() => setShowUpgradeModal(false)} className="mb-4 text-purple-700">← Back</button>
          
          <div className="bg-white rounded-3xl shadow-2xl overflow-hidden">
            <div className="bg-gradient-to-r from-purple-600 to-pink-600 p-8 text-white text-center">
              <Crown className="w-16 h-16 mx-auto mb-4" />
              <h2 className="text-4xl font-bold mb-2">Upgrade to Premium</h2>
            </div>

            <div className="p-8">
              <div className="grid md:grid-cols-2 gap-6 mb-8">
                <div className="border-2 rounded-2xl p-6">
                  <h3 className="text-2xl font-bold mb-4">Free</h3>
                  <p className="text-3xl font-bold mb-6">$0/mo</p>
                  <ul className="space-y-3 text-sm">
                    <li>✓ 3 plants max</li>
                    <li>✓ 2 diagnoses/month</li>
                    <li>✓ 5 photos/plant</li>
                  </ul>
                </div>

                <div className="border-4 border-purple-500 rounded-2xl p-6 bg-purple-50">
                  <h3 className="text-2xl font-bold mb-4">Premium</h3>
                  <p className="text-3xl font-bold text-purple-600 mb-2">$4.99/mo</p>
                  <p className="text-sm mb-6">or $39.99/yr (33% off)</p>
                  <ul className="space-y-3 text-sm mb-6 font-medium">
                    <li>★ Unlimited plants</li>
                    <li>★ Unlimited diagnoses</li>
                    <li>★ Unlimited photos</li>
                    <li>★ Priority support</li>
                  </ul>
                  <button 
                    onClick={async () => {
                      const upd = { ...user, isPremium: true };
                      await storage.set(`user:${user.email}`, JSON.stringify(upd));
                      await storage.set('current-user', JSON.stringify(upd));
                      setIsPremium(true);
                      setShowUpgradeModal(false);
                    }}
                    className="w-full bg-gradient-to-r from-purple-600 to-pink-600 text-white font-bold py-4 rounded-xl"
                  >
                    Upgrade Now (Demo)
                  </button>
                </div>
              </div>

              <button onClick={() => setShowUpgradeModal(false)} className="w-full text-gray-600">
                Continue with Free
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }

  if (screen === 'home') {
    const overdue = getOverdue();
    
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 p-6">
        <div className="max-w-2xl mx-auto">
          <div className="flex justify-between mb-8">
            <div>
              <Leaf className="w-12 h-12 text-green-600 mb-2" />
              <h1 className="text-3xl font-bold">Plant Health</h1>
              <p className="text-sm text-gray-600">{user?.email}</p>
            </div>
            <button 
              onClick={handleLogout} 
              className="flex items-center gap-2 bg-red-600 text-white px-4 py-2 rounded-lg h-fit"
            >
              <LogOut className="w-4 h-4" />Logout
            </button>
          </div>

          <div className="text-center mb-6">
            {isPremium ? (
              <div className="inline-flex items-center gap-2 bg-gradient-to-r from-yellow-400 to-orange-500 text-white px-6 py-2 rounded-full font-semibold">
                <Crown className="w-5 h-5" />Premium
              </div>
            ) : (
              <button 
                onClick={() => setShowUpgradeModal(true)} 
                className="inline-flex items-center gap-2 bg-gradient-to-r from-purple-600 to-pink-600 text-white px-6 py-2 rounded-full font-semibold"
              >
                <Crown className="w-5 h-5" />Upgrade
              </button>
            )}
          </div>

          {!isPremium && (
            <div className="bg-white rounded-xl shadow p-4 mb-6">
              <div className="flex justify-between text-sm">
                <span>Plants: {savedPlants.length}/3</span>
                <span>Diagnoses: {usageStats.diagnosesThisMonth}/2</span>
              </div>
            </div>
          )}

          {overdue.length > 0 && (
            <div className="bg-red-50 border-l-4 border-red-500 rounded-lg p-4 mb-6">
              <div className="flex gap-3">
                <Bell className="w-6 h-6 text-red-600" />
                <div>
                  <h3 className="font-semibold text-red-800">
                    {overdue.length} Plant{overdue.length > 1 ? 's' : ''} Need Water!
                  </h3>
                  {overdue.map(p => (
                    <p key={p.id} className="text-sm text-red-700">{p.name}</p>
                  ))}
                </div>
              </div>
            </div>
          )}

          <div className="bg-white rounded-2xl shadow-xl p-8 mb-6">
            <h2 className="text-2xl font-semibold mb-6">Diagnose Your Plant</h2>
            <input 
              type="text" 
              value={plantName} 
              onChange={(e) => setPlantName(e.target.value)} 
              placeholder="Plant name (optional)" 
              className="w-full px-4 py-3 border rounded-lg mb-4 focus:ring-2 focus:ring-green-500" 
            />
            <input 
              type="file" 
              accept="image/*" 
              onChange={handleImageUpload} 
              className="hidden" 
              id="img" 
            />
            <label 
              htmlFor="img" 
              className="flex items-center justify-center gap-3 w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-4 rounded-lg cursor-pointer transition-colors shadow-lg"
            >
              <Camera className="w-5 h-5" />Upload Plant Photo
            </label>
            <p className="text-center text-sm text-gray-500 mt-4">Get AI-powered diagnosis in seconds</p>
          </div>

          {storageLoading ? (
            <div className="bg-white rounded-2xl shadow-xl p-8 text-center">
              <Leaf className="w-12 h-12 text-green-600 animate-pulse mx-auto mb-4" />
              <p>Loading...</p>
            </div>
          ) : savedPlants.length > 0 ? (
            <div className="bg-white rounded-2xl shadow-xl p-8">
              <h2 className="text-2xl font-semibold mb-4">My Plants</h2>
              <div className="space-y-4">
                {savedPlants.map(p => {
                  const ws = getWaterStatus(p.wateringSchedule.nextWatering);
                  return (
                    <div 
                      key={p.id} 
                      onClick={() => { setSelectedPlant(p); setScreen('detail'); }} 
                      className="flex items-center gap-4 p-4 bg-gray-50 rounded-lg hover:bg-gray-100 cursor-pointer"
                    >
                      <img 
                        src={p.progressPhotos[p.progressPhotos.length - 1].image} 
                        alt={p.name} 
                        className="w-20 h-20 object-cover rounded-lg" 
                      />
                      <div className="flex-1">
                        <h3 className="font-semibold">{p.name}</h3>
                        <p className="text-sm text-gray-600">{p.diagnosis.primaryDiagnosis}</p>
                        <div className={`inline-flex items-center gap-1 px-2 py-1 rounded text-xs ${ws.bg} ${ws.color} mt-1`}>
                          <Droplets className="w-3 h-3" />{ws.text}
                        </div>
                      </div>
                      <ChevronRight className="w-5 h-5 text-gray-400" />
                    </div>
                  );
                })}
              </div>
            </div>
          ) : (
            <div className="bg-white rounded-2xl shadow-xl p-8 text-center">
              <Leaf className="w-12 h-12 text-gray-400 mx-auto mb-4" />
              <p>No plants yet!</p>
            </div>
          )}
        </div>
      </div>
    );
  }

  if (screen === 'detail' && selectedPlant) {
    const ws = getWaterStatus(selectedPlant.wateringSchedule.nextWatering);
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 p-6">
        <div className="max-w-2xl mx-auto">
          <button 
            onClick={() => { setSelectedPlant(null); setScreen('home'); }} 
            className="mb-4 text-green-700"
          >
            ← Back
          </button>
          <div className="bg-white rounded-2xl shadow-xl p-8">
            <div className="flex justify-between mb-6">
              <div>
                <h1 className="text-3xl font-bold">{selectedPlant.name}</h1>
                <p className="text-gray-600">{selectedPlant.diagnosis.primaryDiagnosis}</p>
              </div>
              <button 
                onClick={() => delPlant(selectedPlant.id)} 
                className="text-red-600"
              >
                <X className="w-6 h-6" />
              </button>
            </div>

            <div className={`${ws.bg} rounded-lg p-6 mb-6`}>
              <div className="flex justify-between mb-4">
                <div>
                  <h3 className="font-semibold">Watering</h3>
                  <p className={`text-sm ${ws.color}`}>{ws.text}</p>
                </div>
                <button 
                  onClick={() => markWatered(selectedPlant.id)} 
                  className="bg-blue-600 text-white px-4 py-2 rounded-lg text-sm"
                >
                  Mark Watered
                </button>
              </div>
              <p className="text-sm text-gray-600">
                Last: {new Date(selectedPlant.wateringSchedule.lastWatered).toLocaleDateString()}
              </p>
            </div>

            <div className="mb-6">
              <div className="flex justify-between mb-4">
                <h3 className="text-xl font-semibold flex items-center gap-2">
                  <TrendingUp className="w-5 h-5" />Recovery Progress
                </h3>
                <label className="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg text-sm cursor-pointer flex items-center gap-1 transition-colors">
                  <Plus className="w-4 h-4" />Add Photo
                  <input 
                    type="file" 
                    accept="image/*" 
                    className="hidden" 
                    onChange={(e) => {
                      const file = e.target.files[0];
                      if (file) {
                        const reader = new FileReader();
                        reader.onloadend = () => {
                          const notes = window.prompt ? window.prompt('Add notes (optional):') : 'Progress update';
                          addPhoto(selectedPlant.id, reader.result, notes);
                        };
                        reader.readAsDataURL(file);
                      }
                    }} 
                  />
                </label>
              </div>
              
              <div className="space-y-4">
                {selectedPlant.progressPhotos.slice().reverse().map((photo, idx) => (
                  <div key={idx} className="flex gap-4 bg-gray-50 rounded-lg p-4">
                    <img 
                      src={photo.image} 
                      alt="Progress" 
                      className="w-24 h-24 object-cover rounded-lg" 
                    />
                    <div>
                      <p className="text-sm text-gray-600">
                        {new Date(photo.date).toLocaleDateString()}
                      </p>
                      <p className="text-sm">{photo.notes}</p>
                    </div>
                  </div>
                ))}
              </div>
            </div>

            <div className="border-t pt-6">
              <h3 className="text-xl font-semibold mb-4">Diagnosis</h3>
              <div className="bg-green-50 rounded-lg p-4 mb-4">
                <h4 className="font-semibold mb-2">Treatment</h4>
                <ol className="space-y-2 text-sm">
                  {selectedPlant.diagnosis.treatment.map((step, i) => (
                    <li key={i}>{i + 1}. {step}</li>
                  ))}
                </ol>
              </div>
              <div className="bg-blue-50 rounded-lg p-4">
                <h4 className="font-semibold mb-2">Timeline</h4>
                <p className="text-sm">{selectedPlant.diagnosis.timeline}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    );
  }

  if (screen === 'questionnaire') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 p-6">
        <div className="max-w-2xl mx-auto">
          <button 
            onClick={() => { setScreen('home'); setImageData(null); }} 
            className="mb-4 text-green-700"
          >
            ← Back
          </button>
          <div className="bg-white rounded-2xl shadow-xl p-8">
            <h2 className="text-2xl font-semibold mb-6">Plant Care Context</h2>
            
            {imageData && (
              <img 
                src={imageData} 
                alt="Plant" 
                className="w-full h-64 object-cover rounded-lg mb-6" 
              />
            )}

            <div className="space-y-6">
              <div>
                <label className="flex items-center gap-2 font-medium mb-2">
                  <Droplets className="w-5 h-5 text-blue-500" />
                  When last watered?
                </label>
                <select 
                  value={questionnaire.lastWatered} 
                  onChange={(e) => setQuestionnaire(p => ({...p, lastWatered: e.target.value}))} 
                  className="w-full px-4 py-3 border rounded-lg"
                >
                  <option value="">Select...</option>
                  <option value="Today">Today</option>
                  <option value="1-2 days ago">1-2 days ago</option>
                  <option value="3-5 days ago">3-5 days ago</option>
                  <option value="1 week ago">1 week ago</option>
                  <option value="2+ weeks ago">2+ weeks ago</option>
                </select>
              </div>

              <div>
                <label className="flex items-center gap-2 font-medium mb-2">
                  <Sun className="w-5 h-5 text-yellow-500" />
                  Light Condition
                </label>
                <select 
                  value={questionnaire.lightCondition} 
                  onChange={(e) => setQuestionnaire(p => ({...p, lightCondition: e.target.value}))} 
                  className="w-full px-4 py-3 border rounded-lg"
                >
                  <option value="">Select...</option>
                  <option value="Direct sunlight">Direct sunlight</option>
                  <option value="Bright indirect">Bright indirect</option>
                  <option value="Medium light">Medium light</option>
                  <option value="Low light">Low light</option>
                </select>
              </div>

              <div>
                <label className="font-medium mb-2 block">Soil Condition</label>
                <select 
                  value={questionnaire.soilCondition} 
                  onChange={(e) => setQuestionnaire(p => ({...p, soilCondition: e.target.value}))} 
                  className="w-full px-4 py-3 border rounded-lg"
                >
                  <option value="">Select...</option>
                  <option value="Bone dry">Bone dry</option>
                  <option value="Slightly moist">Slightly moist</option>
                  <option value="Very wet">Very wet</option>
                  <option value="Soggy/waterlogged">Soggy/waterlogged</option>
                </select>
              </div>

              <div>
                <label className="font-medium mb-2 block">Recent Changes</label>
                <input 
                  type="text" 
                  value={questionnaire.recentChanges} 
                  onChange={(e) => setQuestionnaire(p => ({...p, recentChanges: e.target.value}))} 
                  placeholder="e.g., Repotted last week" 
                  className="w-full px-4 py-3 border rounded-lg" 
                />
              </div>

              <div>
                <label className="font-medium mb-3 block">Symptoms</label>
                <div className="grid grid-cols-2 gap-2">
                  {symptoms.map(s => (
                    <button 
                      key={s} 
                      onClick={() => toggleSymptom(s)} 
                      className={`px-4 py-2 rounded-lg text-sm ${
                        questionnaire.symptoms.includes(s) 
                          ? 'bg-green-600 text-white' 
                          : 'bg-gray-100 text-gray-700'
                      }`}
                    >
                      {s}
                    </button>
                  ))}
                </div>
              </div>

              <button 
                onClick={getDiagnosis} 
                disabled={!questionnaire.lastWatered || !questionnaire.soilCondition || !questionnaire.lightCondition} 
                className="w-full bg-green-600 hover:bg-green-700 disabled:bg-gray-300 text-white font-semibold py-4 rounded-lg flex items-center justify-center gap-2"
              >
                Get Diagnosis<ChevronRight className="w-5 h-5" />
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }

  if (screen === 'diagnosis') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 p-6">
        <div className="max-w-2xl mx-auto">
          <button 
            onClick={() => setScreen('home')} 
            className="mb-4 text-green-700"
          >
            ← Back
          </button>
          <div className="bg-white rounded-2xl shadow-xl p-8">
            {loading ? (
              <div className="text-center py-12">
                <Leaf className="w-16 h-16 text-green-600 animate-pulse mx-auto mb-4" />
                <p className="text-xl">Analyzing your plant...</p>
              </div>
            ) : diagnosis && (
              <div>
                <h2 className="text-3xl font-bold mb-2">{diagnosis.primaryDiagnosis}</h2>
                <div className="flex items-center gap-2 mb-4">
                  {diagnosis.confidence === 'high' ? (
                    <CheckCircle className="w-5 h-5 text-green-600" />
                  ) : (
                    <AlertCircle className="w-5 h-5 text-yellow-600" />
                  )}
                  <span className="text-sm font-medium">
                    {diagnosis.confidence.charAt(0).toUpperCase() + diagnosis.confidence.slice(1)} Confidence
                  </span>
                </div>
                {imageData && (
                  <img 
                    src={imageData} 
                    alt="Plant" 
                    className="w-full h-48 object-cover rounded-lg mb-4" 
                  />
                )}
                <p className="text-gray-700 mb-6">{diagnosis.explanation}</p>
                <div className="space-y-4 mb-6">
                  <div className="bg-red-50 rounded-lg p-4">
                    <h3 className="font-semibold mb-2 flex items-center gap-2">
                      <AlertCircle className="w-5 h-5 text-red-600" />
                      Causes
                    </h3>
                    <ul className="space-y-1 text-sm">
                      {diagnosis.causes.map((c, i) => (
                        <li key={i}>• {c}</li>
                      ))}
                    </ul>
                  </div>
                  <div className="bg-green-50 rounded-lg p-4">
                    <h3 className="font-semibold mb-2 flex items-center gap-2">
                      <CheckCircle className="w-5 h-5 text-green-600" />
                      Treatment
                    </h3>
                    <ol className="space-y-2 text-sm">
                      {diagnosis.treatment.map((t, i) => (
                        <li key={i} className="flex gap-2">
                          <span className="bg-green-600 text-white rounded-full w-5 h-5 flex items-center justify-center text-xs font-bold">
                            {i+1}
                          </span>
                          {t}
                        </li>
                      ))}
                    </ol>
                  </div>
                  <div className="bg-blue-50 rounded-lg p-4">
                    <h3 className="font-semibold mb-2">Timeline</h3>
                    <p className="text-sm">{diagnosis.timeline}</p>
                  </div>
                </div>
                <button 
                  onClick={savePlant} 
                  className="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-4 rounded-lg"
                >
                  Save Plant
                </button>
              </div>
            )}
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 p-6 flex items-center justify-center">
      <Leaf className="w-16 h-16 text-green-600 animate-pulse" />
    </div>
  );
}
```
