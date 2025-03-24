# VIAD test 2
import React, { useState, useEffect } from 'react';
import { ArrowRight, Info, HelpCircle, ChevronDown, Check, X, Edit2 } from 'lucide-react';

// Composant principal
const ViagerSimulator = () => {
  // États pour stocker les données du formulaire
  const [step, setStep] = useState(1);
  const [propertyValue, setPropertyValue] = useState(300000);
  const [manualPropertyValue, setManualPropertyValue] = useState('300000');
  const [age, setAge] = useState(75);
  const [manualAge, setManualAge] = useState('75');
  const [gender, setGender] = useState('female');
  const [occupationType, setOccupationType] = useState('occupied');
  const [bouquetPercentage, setBouquetPercentage] = useState(30);
  const [manualBouquetPercentage, setManualBouquetPercentage] = useState('30');
  const [editMode, setEditMode] = useState({
    property: false,
    age: false,
    bouquet: false
  });
  const [results, setResults] = useState(null);
  const [showTooltip, setShowTooltip] = useState(null);
  const [isCalculating, setIsCalculating] = useState(false);
  const [showSuccess, setShowSuccess] = useState(false);
  
  // Table de mortalité INSEE 2020-2022
  const lifeExpectancyTable = {
    male: {
      65: 19.02,
      70: 15.41,
      75: 11.99,
      80: 8.84,
      85: 6.11,
      90: 4.01,
      95: 2.68,
      99: 2.15,
      100: 2.09
    },
    female: {
      65: 22.99,
      70: 18.79,
      75: 14.74,
      80: 10.97,
      85: 7.67,
      90: 5.04,
      95: 3.27,
      99: 2.40,
      100: 2.23
    }
  };

  // Fonction pour interpoler linéairement entre les valeurs
  const interpolateLifeExpectancy = (gender, age) => {
    const table = lifeExpectancyTable[gender];
    const ages = Object.keys(table).map(Number).sort((a, b) => a - b);
    
    // Si l'âge est dans le tableau, retourner la valeur exacte
    if (table[age] !== undefined) {
      return table[age];
    }
    
    // Si l'âge est inférieur au minimum du tableau, utiliser la valeur minimum
    if (age < ages[0]) {
      return table[ages[0]];
    }
    
    // Si l'âge est supérieur au maximum du tableau, utiliser la valeur maximum
    if (age > ages[ages.length - 1]) {
      return table[ages[ages.length - 1]];
    }
    
    // Trouver les deux âges qui encadrent l'âge demandé
    let lowerAge = ages[0];
    let upperAge = ages[ages.length - 1];
    
    for (let i = 0; i < ages.length - 1; i++) {
      if (age >= ages[i] && age <= ages[i + 1]) {
        lowerAge = ages[i];
        upperAge = ages[i + 1];
        break;
      }
    }
    
    // Interpolation linéaire
    const ratio = (age - lowerAge) / (upperAge - lowerAge);
    return table[lowerAge] + ratio * (table[upperAge] - table[lowerAge]);
  };

  // Calcul du viager
  const calculateViager = () => {
    setIsCalculating(true);
    
    // Simulation d'un appel API avec setTimeout
    setTimeout(() => {
      // Espérance de vie basée sur les tables INSEE 2020-2022
      const lifeExpectancy = Math.round(interpolateLifeExpectancy(gender, age) * 10) / 10;
      
      // Calcul du DUH (Droit d'Usage et d'Habitation)
      const duhPercentage = occupationType === 'occupied' 
        ? Math.min(70, 35 + age * 0.5)
        : 0;
      
      // Valeur vénale ajustée
      const adjustedValue = propertyValue * (1 - duhPercentage / 100);
      
      // Bouquet
      const bouquet = Math.round(adjustedValue * (bouquetPercentage / 100));
      
      // Rente mensuelle
      const remainingValue = adjustedValue - bouquet;
      const monthlyPayment = Math.round(remainingValue / (lifeExpectancy * 12));
      
      // Rentabilité annuelle estimée
      const annualReturn = Math.round((monthlyPayment * 12) / bouquet * 100 * 10) / 10;
      
      setResults({
        propertyValue,
        duhPercentage,
        adjustedValue,
        bouquet,
        monthlyPayment,
        lifeExpectancy,
        annualReturn
      });
      
      setIsCalculating(false);
      setStep(3);
    }, 1500);
  };
  
  // Fonction pour formater les valeurs monétaires
  const formatMoney = (amount) => {
    return new Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'EUR' }).format(amount);
  };
  
  // Fonction pour passer à l'étape suivante
  const nextStep = () => {
    if (step === 2) {
      calculateViager();
    } else {
      setStep(step + 1);
    }
  };
  
  // Fonction pour revenir à l'étape précédente
  const prevStep = () => {
    setStep(step - 1);
  };

  // Gestion des entrées manuelles
  const handleManualPropertyChange = (e) => {
    const value = e.target.value.replace(/\D/g, '');
    setManualPropertyValue(value);
  };

  const handleManualAgeChange = (e) => {
    const value = e.target.value.replace(/\D/g, '');
    setManualAge(value);
  };

  const handleManualBouquetChange = (e) => {
    const value = e.target.value.replace(/\D/g, '');
    setManualBouquetPercentage(value);
  };

  const applyManualProperty = () => {
    const value = Number(manualPropertyValue);
    if (value >= 50000 && value <= 2000000) {
      setPropertyValue(value);
    } else {
      setManualPropertyValue(propertyValue.toString());
    }
    setEditMode({...editMode, property: false});
  };

  const applyManualAge = () => {
    const value = Number(manualAge);
    if (value >= 60 && value <= 100) {
      setAge(value);
    } else {
      setManualAge(age.toString());
    }
    setEditMode({...editMode, age: false});
  };

  const applyManualBouquet = () => {
    const value = Number(manualBouquetPercentage);
    if (value >= 10 && value <= 70) {
      setBouquetPercentage(value);
    } else {
      setManualBouquetPercentage(bouquetPercentage.toString());
    }
    setEditMode({...editMode, bouquet: false});
  };
  
  // Animation lors du calcul terminé
  useEffect(() => {
    if (results && step === 3) {
      const timer = setTimeout(() => {
        setShowSuccess(true);
        setTimeout(() => setShowSuccess(false), 3000);
      }, 500);
      return () => clearTimeout(timer);
    }
  }, [results, step]);
  
  // Composant de tooltip
  const Tooltip = ({ id, content }) => {
    if (showTooltip !== id) return null;
    
    return (
      <div className="absolute z-10 bg-white rounded-lg shadow-lg p-3 max-w-xs text-sm text-gray-600 border border-gray-200 -mt-1 right-8">
        {content}
        <button 
          className="absolute -top-2 -right-2 bg-gray-100 rounded-full p-1"
          onClick={() => setShowTooltip(null)}
        >
          <X size={12} />
        </button>
      </div>
    );
  };

  // Barre de progression
  const ProgressBar = () => (
    <div className="flex items-center justify-between w-full mb-8 mt-2">
      {[1, 2, 3].map(stepNumber => (
        <div key={stepNumber} className="flex flex-col items-center">
          <div 
            className={`w-8 h-8 flex items-center justify-center rounded-full transition-all duration-500 ${
              step >= stepNumber 
                ? 'bg-blue-600 text-white' 
                : 'bg-gray-200 text-gray-500'
            }`}
          >
            {stepNumber}
          </div>
          <span className={`text-xs mt-1 transition-all duration-500 ${
            step >= stepNumber ? 'text-blue-600 font-medium' : 'text-gray-500'
          }`}>
            {stepNumber === 1 ? 'Bien' : stepNumber === 2 ? 'Vendeur' : 'Résultats'}
          </span>
        </div>
      ))}
    </div>
  );

  // Composant pour l'entrée manuelle ou le curseur
  const InputWithSlider = ({ 
    label, 
    value, 
    min, 
    max, 
    step, 
    onChange, 
    formatValue, 
    isEditing, 
    setIsEditing,
    manualValue,
    onManualChange,
    applyManual,
    tooltipId,
    tooltipContent
  }) => (
    <div>
      <div className="flex items-center justify-between mb-2">
        <label className="text-gray-700 font-medium">{label}</label>
        {tooltipId && (
          <div className="relative">
            <button 
              onClick={() => setShowTooltip(tooltipId)}
              className="text-gray-400 hover:text-gray-600"
            >
              <HelpCircle size={16} />
            </button>
            <Tooltip id={tooltipId} content={tooltipContent} />
          </div>
        )}
      </div>

      <div className="flex justify-between mb-2">
        {isEditing ? (
          <div className="w-full flex">
            <input
              type="text"
              value={manualValue}
              onChange={onManualChange}
              className="flex-1 border border-blue-300 rounded-l-lg p-2 text-right focus:outline-none focus:ring-2 focus:ring-blue-500"
              autoFocus
            />
            <button
              onClick={applyManual}
              className="bg-blue-600 text-white px-3 rounded-r-lg hover:bg-blue-700"
            >
              <Check size={16} />
            </button>
          </div>
        ) : (
          <>
            <button
              onClick={() => setIsEditing(true)}
              className="text-blue-600 hover:text-blue-800 flex items-center"
            >
              <Edit2 size={14} className="mr-1" />
              <span className="text-xs">Modifier</span>
            </button>
            <div className="text-blue-600 font-bold">{formatValue(value)}</div>
          </>
        )}
      </div>

      <input 
        type="range" 
        min={min} 
        max={max} 
        step={step} 
        value={value} 
        onChange={onChange}
        className="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-600"
      />
      <div className="flex justify-between mt-1 text-xs text-gray-500">
        <span>{formatValue(min)}</span>
        <span>{formatValue(max)}</span>
      </div>
    </div>
  );
  
  return (
    <div className="min-h-screen bg-white p-4 flex flex-col items-center justify-center font-sans">
      <div className="w-full max-w-3xl bg-white rounded-xl shadow-lg overflow-hidden transition-all duration-500 transform">
        <div className="bg-gradient-to-r from-blue-600 to-blue-800 text-white p-6">
          <h1 className="text-2xl font-bold mb-1">Simulateur de Viager</h1>
          <p className="text-blue-100">Calculez rapidement et précisément les modalités d'un viager</p>
        </div>
        
        <div className="p-6">
          <ProgressBar />
          
          {/* Étape 1: Informations sur le bien */}
          {step === 1 && (
            <div className="space-y-6 animate-fadeIn">
              <h2 className="text-xl font-semibold text-gray-800 mb-4">Informations sur le bien</h2>
              
              <div className="space-y-6">
                <InputWithSlider 
                  label="Valeur du bien"
                  value={propertyValue}
                  min={50000}
                  max={2000000}
                  step={10000}
                  onChange={(e) => setPropertyValue(Number(e.target.value))}
                  formatValue={(val) => formatMoney(val)}
                  isEditing={editMode.property}
                  setIsEditing={() => setEditMode({...editMode, property: true})}
                  manualValue={manualPropertyValue}
                  onManualChange={handleManualPropertyChange}
                  applyManual={applyManualProperty}
                />
                
                <div>
                  <div className="flex items-center justify-between mb-3">
                    <label className="text-gray-700 font-medium">Type d'occupation</label>
                    <div className="relative">
                      <button 
                        onClick={() => setShowTooltip('occupation')}
                        className="text-gray-400 hover:text-gray-600"
                      >
                        <HelpCircle size={16} />
                      </button>
                      <Tooltip 
                        id="occupation" 
                        content="Viager occupé: le vendeur continue à habiter le bien. Viager libre: l'acheteur peut occuper ou louer le bien immédiatement."
                      />
                    </div>
                  </div>
                  <div className="grid grid-cols-2 gap-3">
                    <button
                      onClick={() => setOccupationType('occupied')}
                      className={`p-4 border-2 rounded-lg transition-all flex items-center justify-center ${
                        occupationType === 'occupied' 
                          ? 'border-blue-600 bg-blue-50 text-blue-600' 
                          : 'border-gray-300 hover:bg-gray-50'
                      }`}
                    >
                      {occupationType === 'occupied' && <Check size={18} className="mr-2 text-blue-600" />}
                      <div className="flex flex-col items-center">
                        <span className="font-medium">Viager occupé</span>
                        <span className="text-xs text-gray-500 mt-1">Le vendeur reste dans le bien</span>
                      </div>
                    </button>
                    <button
                      onClick={() => setOccupationType('free')}
                      className={`p-4 border-2 rounded-lg transition-all flex items-center justify-center ${
                        occupationType === 'free' 
                          ? 'border-blue-600 bg-blue-50 text-blue-600' 
                          : 'border-gray-300 hover:bg-gray-50'
                      }`}
                    >
                      {occupationType === 'free' && <Check size={18} className="mr-2 text-blue-600" />}
                      <div className="flex flex-col items-center">
                        <span className="font-medium">Viager libre</span>
                        <span className="text-xs text-gray-500 mt-1">Libre d'occupation</span>
                      </div>
                    </button>
                  </div>
                </div>
              </div>
              
              <div className="pt-6">
                <button
                  onClick={nextStep}
                  className="w-full bg-blue-600 text-white p-4 rounded-lg hover:bg-blue-700 transition flex items-center justify-center shadow-md"
                >
                  Continuer
                  <ArrowRight size={18} className="ml-2" />
                </button>
              </div>

              <div className="mt-6 text-center">
                <div className="inline-block animate-bounce bg-orange-500 text-white px-4 py-2 rounded-full text-sm">
                  Simulez votre viager en quelques clics !
                </div>
              </div>
            </div>
          )}
          
          {/* Étape 2: Informations sur le crédirentier (vendeur) */}
          {step === 2 && (
            <div className="space-y-6 animate-fadeIn">
              <h2 className="text-xl font-semibold text-gray-800 mb-4">Informations sur le vendeur</h2>
              
              <div className="space-y-6">
                <InputWithSlider 
                  label="Âge du crédirentier"
                  value={age}
                  min={60}
                  max={100}
                  step={1}
                  onChange={(e) => setAge(Number(e.target.value))}
                  formatValue={(val) => `${val} ans`}
                  isEditing={editMode.age}
                  setIsEditing={() => setEditMode({...editMode, age: true})}
                  manualValue={manualAge}
                  onManualChange={handleManualAgeChange}
                  applyManual={applyManualAge}
                />
                
                <div>
                  <label className="text-gray-700 font-medium block mb-3">Genre</label>
                  <div className="grid grid-cols-2 gap-4">
                    <button
                      onClick={() => setGender('male')}
                      className={`p-4 border-2 rounded-lg transition-all flex items-center justify-center ${
                        gender === 'male' 
                          ? 'border-blue-600 bg-blue-50 text-blue-600' 
                          : 'border-gray-300 hover:bg-gray-50'
                      }`}
                    >
                      <div className="flex flex-col items-center">
                        <div className="w-16 h-16 rounded-full bg-blue-100 mb-2 flex items-center justify-center">
                          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="text-blue-600">
                            <circle cx="12" cy="7" r="4"/>
                            <path d="M12 11v8"/>
                            <path d="M9 18h6"/>
                          </svg>
                        </div>
                        <span className="font-medium">Homme</span>
                      </div>
                    </button>
                    <button
                      onClick={() => setGender('female')}
                      className={`p-4 border-2 rounded-lg transition-all flex items-center justify-center ${
                        gender === 'female' 
                          ? 'border-blue-600 bg-blue-50 text-blue-600' 
                          : 'border-gray-300 hover:bg-gray-50'
                      }`}
                    >
                      <div className="flex flex-col items-center">
                        <div className="w-16 h-16 rounded-full bg-blue-100 mb-2 flex items-center justify-center">
                          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className="text-blue-600">
                            <circle cx="12" cy="7" r="4"/>
                            <path d="M9 20V14H15V20"/>
                            <path d="M12 14v4"/>
                          </svg>
                        </div>
                        <span className="font-medium">Femme</span>
                      </div>
                    </button>
                  </div>
                </div>
                
                <InputWithSlider 
                  label="Pourcentage du bouquet"
                  value={bouquetPercentage}
                  min={10}
                  max={70}
                  step={1}
                  onChange={(e) => setBouquetPercentage(Number(e.target.value))}
                  formatValue={(val) => `${val}%`}
                  isEditing={editMode.bouquet}
                  setIsEditing={() => setEditMode({...editMode, bouquet: true})}
                  manualValue={manualBouquetPercentage}
                  onManualChange={handleManualBouquetChange}
                  applyManual={applyManualBouquet}
                  tooltipId="bouquet"
                  tooltipContent="Le bouquet est le capital versé au comptant au moment de la signature de l'acte de vente. Le reste sera versé sous forme de rente."
                />
              </div>
              
              <div className="pt-6 flex space-x-3">
                <button
                  onClick={prevStep}
                  className="flex-1 border-2 border-gray-300 text-gray-700 p-4 rounded-lg hover:bg-gray-50 transition"
                >
                  Retour
                </button>
                <button
                  onClick={nextStep}
                  className="flex-1 bg-blue-600 text-white p-4 rounded-lg hover:bg-blue-700 transition flex items-center justify-center shadow-md"
                >
                  {isCalculating ? (
                    <span className="flex items-center">
                      <svg className="animate-spin -ml-1 mr-2 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                        <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                      </svg>
                      Calcul en cours...
                    </span>
                  ) : (
                    <span className="flex items-center">
                      Calculer
                      <ArrowRight size={18} className="ml-2" />
                    </span>
                  )}
                </button>
              </div>
            </div>
          )}
          
          {/* Étape 3: Résultats */}
          {step === 3 && results && (
            <div className="space-y-6 animate-fadeIn">
              <div className={`fixed top-12 right-12 bg-green-100 border border-green-200 text-green-800 px-4 py-3 rounded-lg flex items-center transform transition-all duration-500 ${showSuccess ? 'opacity-100 translate-y-0' : 'opacity-0 -translate-y-4'}`}>
                <Check size={20} className="mr-2" />
                Simulation terminée avec succès !
              </div>
              
              <h2 className="text-xl font-semibold text-gray-800 mb-4">Résultats de la simulation</h2>
              
              <div className="grid md:grid-cols-2 gap-5">
                <div className="bg-white p-6 rounded-lg border-2 border-blue-200 shadow-md transform hover:scale-105 transition-all duration-300">
                  <div className="text-sm text-blue-600 font-medium mb-2">Bouquet</div>
                  <div className="text-3xl font-bold text-gray-800">{formatMoney(results.bouquet)}</div>
                  <div className="text-sm text-gray-500 mt-2">Capital initial versé</div>
                  <div className="w-full h-2 bg-gray-100 rounded-full mt-4">
                    <div 
                      className="h-2 bg-blue-600 rounded-full" 
                      style={{ width: `${bouquetPercentage}%` }}
                    ></div>
                  </div>
                  <div className="text-xs text-gray-500 mt-1 text-right">{bouquetPercentage}% de la valeur</div>
                </div>
                
                <div className="bg-white p-6 rounded-lg border-2 border-orange-200 shadow-md transform hover:scale-105 transition-all duration-300">
                  <div className="text-sm text-orange-500 font-medium mb-2">Rente mensuelle</div>
                  <div className="text-3xl font-bold text-gray-800">{formatMoney(results.monthlyPayment)}</div>
                  <div className="text-sm text-gray-500 mt-2">Versement périodique</div>
                  <div className="flex items-end justify-between mt-4">
                    <div className="bg-orange-100 h-4 w-4 rounded"></div>
                    <div className="bg-orange-200 h-8 w-4 rounded"></div>
                    <div className="bg-orange-300 h-12 w-4 rounded"></div>
                    <div className="bg-orange-400 h-16 w-4 rounded"></div>
                    <div className="bg-orange-500 h-20 w-4 rounded"></div>
                  </div>
                  <div className="text-xs text-gray-500 mt-1 text-right">
                    Pendant environ {results.lifeExpectancy} ans
                  </div>
                  <div className="bg-blue-50 text-xs rounded p-2 mt-3 border border-blue-100">
                    Basé sur les tables de mortalité INSEE 2020-2022
                  </div>
                </div>
              </div>
              
              <div className="bg-blue-50 p-6 rounded-lg border border-blue-100 shadow-md">
                <h3 className="font-medium text-blue-800 mb-4 flex items-center">
                  <Info size={18} className="mr-2" />
                  Détails du calcul
                </h3>
                
                <div className="space-y-4">
                  <div className="flex items-center justify-between">
                    <span className="text-gray-700">Valeur du bien</span>
                    <span className="font-medium text-gray-900">{formatMoney(results.propertyValue)}</span>
                  </div>
                  
                  {occupationType === 'occupied' && (
                    <>
                      <div className="flex items-center justify-between">
                        <div className="flex items-center">
                          <span className="text-gray-700">Décote DUH</span>
                          <button 
                            onClick={() => setShowTooltip('duh')}
                            className="text-gray-400 hover:text-gray-600 ml-1"
                          >
                            <HelpCircle size={14} />
                          </button>
                          <Tooltip 
                            id="duh" 
                            content="Le Droit d'Usage et d'Habitation représente la valeur économique du droit du vendeur à continuer d'occuper le logement."
                          />
                        </div>
                        <div className="flex items-center">
                          <div className="w-16 bg-gray-200 h-2 rounded-full mr-2">
                            <div 
                              className="bg-orange-500 h-2 rounded-full" 
                              style={{ width: `${results.duhPercentage}%` }}
                            ></div>
                          </div>
                          <span className="font-medium text-gray-900">{results.duhPercentage}%</span>
                        </div>
                      </div>
                      
                      <div className="flex items-center justify-between">
                        <span className="text-gray-700">Valeur occupée</span>
                        <span className="font-medium text-gray-900">{formatMoney(results.adjustedValue)}</span>
                      </div>
                    </>
                  )}
                  
                  <div className="flex items-center justify-between">
                    <div className="flex items-center">
                      <span className="text-gray-700">Espérance de vie restante</span>
                      <button 
                        onClick={() => setShowTooltip('lifeExpectancy')}
                        className="text-gray-400 hover:text-gray-600 ml-1"
                      >
                        <HelpCircle size={14} />
                      </button>
                      <Tooltip 
                        id="lifeExpectancy" 
                        content="Basée sur les tables de mortalité INSEE 2020-2022. Pour une personne de cet âge et de ce genre, c'est la durée de vie moyenne restante statistiquement prévue."
                      />
                    </div>
                    <div className="flex items-center">
                      <div className="flex space-x-1 mr-2">
                        {[...Array(5)].map((_, i) => (
                          <div 
                            key={i} 
                            className={`w-2 h-${4 + i * 2} rounded-t-sm ${
                              i < Math.ceil(results.lifeExpectancy / 5) 
                                ? 'bg-blue-500' 
                                : 'bg-gray-300'
                            }`}
                          ></div>
                        ))}
                      </div>
                      <span className="font-medium text-gray-900">{results.lifeExpectancy} ans</span>
                    </div>
                  </div>
                  
                  <div className="flex items-center justify-between">
                    <span className="text-gray-700">Rentabilité annuelle estimée</span>
                    <span className="font-medium text-orange-500 text-lg">{results.annualReturn}%</span>
                  </div>
                </div>
              </div>
              
              <div className="bg-yellow-50 p-4 rounded-lg border border-yellow-100 flex items-start">
                <Info size={20} className="text-orange-500 mr-3 mt-0.5 flex-shrink-0" />
                <div className="text-sm text-gray-700">
                  Cette simulation est fournie à titre indicatif. Les calculs de viager dépendent de nombreux facteurs et peuvent varier selon les situations. Consultez un notaire pour une estimation précise et des conseils juridiques adaptés.
                </div>
              </div>
              
              <div className="pt-4 flex space-x-3">
                <button
                  onClick={prevStep}
                  className="flex-1 border-2 border-gray-300 text-gray-700 p-4 rounded-lg hover:bg-gray-50 transition"
                >
                  Modifier les paramètres
                </button>
                <button
                  className="flex-1 bg-orange-500 text-white p-4 rounded-lg hover:bg-orange-600 transition flex items-center justify-center shadow-md"
                  onClick={() => {
                    // Ici, on pourrait ajouter une fonction pour exporter les résultats
                    alert("Fonctionnalité d'export en cours de développement");
                  }}
                >
                  Exporter les résultats
                </button>
              </div>
            </div>
          )}
        </div>
      </div>
      
      <div className="mt-6 text-center text-gray-500 text-sm">
        © 2025 Simulateur de Viager Professionnel - Tous droits réservés
      </div>
    </div>
  );
};

// Ajout de styles globaux pour les animations
const style = document.createElement('style');
style.textContent = `
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  
  .animate-fadeIn {
    animation: fadeIn 0.5s ease-out forwards;
  }
`;
document.head.appendChild(style);

export default ViagerSimulator;
