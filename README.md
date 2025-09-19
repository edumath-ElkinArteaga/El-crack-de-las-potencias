# El-crack-de-las-potencias-by-ElkinArteaga
import React, { useState, useEffect, useCallback } from 'react';
import { ChevronRight, Trophy, Star, BookOpen, Lightbulb, CheckCircle, XCircle, Menu, Home } from 'lucide-react';

const PotenciacionGame = () => {
  // Estados principales
  const [gameState, setGameState] = useState('menu');
  const [currentLevel, setCurrentLevel] = useState(0);
  const [currentExercise, setCurrentExercise] = useState(0);
  const [score, setScore] = useState(0);
  const [userAnswer, setUserAnswer] = useState('');
  const [showFeedback, setShowFeedback] = useState(false);
  const [isCorrect, setIsCorrect] = useState(false);
  const [exercises, setExercises] = useState([]);
  const [completedLevels, setCompletedLevels] = useState([]);
  const [hint, setHint] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  // Configuración de niveles
  const levels = [
    {
      title: "Producto de Potencias de Igual Base",
      property: "a^m × a^n = a^(m+n)",
      description: "Cuando multiplicamos potencias que tienen la misma base, mantenemos la base y sumamos los exponentes.",
      tips: [
        "La base permanece igual",
        "Los exponentes se suman",
        "Ejemplo: 2³ × 2⁴ = 2^(3+4) = 2⁷"
      ],
      color: "from-blue-500 to-blue-700"
    },
    {
      title: "Cociente de Potencias de Igual Base", 
      property: "a^m ÷ a^n = a^(m-n)",
      description: "Cuando dividimos potencias que tienen la misma base, mantenemos la base y restamos los exponentes.",
      tips: [
        "La base permanece igual",
        "Los exponentes se restan (minuendo - sustraendo)",
        "Ejemplo: 3⁵ ÷ 3² = 3^(5-2) = 3³"
      ],
      color: "from-green-500 to-green-700"
    },
    {
      title: "Potencia de una Potencia",
      property: "(a^m)^n = a^(m×n)",
      description: "Cuando elevamos una potencia a otra potencia, mantenemos la base y multiplicamos los exponentes.",
      tips: [
        "La base permanece igual",
        "Los exponentes se multiplican",
        "Ejemplo: (2³)² = 2^(3×2) = 2⁶"
      ],
      color: "from-purple-500 to-purple-700"
    },
    {
      title: "Potencia de un Producto",
      property: "(a×b)^n = a^n × b^n",
      description: "Cuando elevamos un producto a una potencia, elevamos cada factor a esa potencia.",
      tips: [
        "Cada factor se eleva a la potencia",
        "Se distribuye el exponente",
        "Ejemplo: (2×3)² = 2² × 3² = 4 × 9"
      ],
      color: "from-orange-500 to-orange-700"
    },
    {
      title: "Potencia de un Cociente",
      property: "(a/b)^n = a^n / b^n",
      description: "Cuando elevamos un cociente a una potencia, elevamos tanto el numerador como el denominador a esa potencia.",
      tips: [
        "El numerador se eleva a la potencia",
        "El denominador se eleva a la potencia",
        "Ejemplo: (3/2)² = 3²/2² = 9/4"
      ],
      color: "from-pink-500 to-pink-700"
    },
    {
      title: "Potencia con Exponente Cero",
      property: "a^0 = 1 (a ≠ 0)",
      description: "Cualquier número diferente de cero elevado a la potencia cero es igual a 1.",
      tips: [
        "Solo números diferentes de cero",
        "El resultado siempre es 1",
        "Ejemplo: 5⁰ = 1, (3/4)⁰ = 1"
      ],
      color: "from-teal-500 to-teal-700"
    },
    {
      title: "Potencia con Exponente Negativo",
      property: "a^(-n) = 1/a^n",
      description: "Una potencia con exponente negativo es igual a 1 dividido por la potencia con exponente positivo.",
      tips: [
        "El exponente negativo invierte la fracción",
        "Se convierte en 1 sobre la potencia positiva",
        "Ejemplo: 2⁻³ = 1/2³ = 1/8"
      ],
      color: "from-red-500 to-red-700"
    },
    {
      title: "Desafío Final - Todas las Propiedades",
      property: "Combinación de todas las propiedades",
      description: "¡Hora del desafío final! Aquí encontrarás ejercicios que combinan todas las propiedades que has aprendido.",
      tips: [
        "Identifica qué propiedad usar",
        "Puedes usar varias propiedades en un ejercicio",
        "¡Confía en tu conocimiento!",
        "Lee cuidadosamente cada ejercicio"
      ],
      color: "from-indigo-500 to-indigo-700"
    }
  ];

  // Persistencia de datos
  useEffect(() => {
    try {
      const saved = JSON.parse(sessionStorage.getItem('potenciacion-progress') || '[]');
      if (Array.isArray(saved)) {
        setCompletedLevels(saved);
      }
    } catch (error) {
      console.error('Error loading progress:', error);
    }
  }, []);

  useEffect(() => {
    try {
      sessionStorage.setItem('potenciacion-progress', JSON.stringify(completedLevels));
    } catch (error) {
      console.error('Error saving progress:', error);
    }
  }, [completedLevels]);

  // Utilidades matemáticas
  const gcd = (a, b) => b === 0 ? Math.abs(a) : gcd(b, a % b);
  const simplifyFraction = (num, den) => {
    if (den === 0) throw new Error('Division by zero');
    const g = gcd(num, den);
    return { num: num / g, den: den / g };
  };
  const getRandomInt = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
  const getRandomFraction = () => {
    const num = getRandomInt(1, 9);
    const den = getRandomInt(2, 9);
    return simplifyFraction(num, den);
  };

  // Validación de respuestas
  const normalizeAnswer = (answer) => {
    if (!answer || typeof answer !== 'string') return '';
    return answer.trim().toLowerCase().replace(/\s+/g, '');
  };

  const validateInput = (input) => {
    const normalized = normalizeAnswer(input);
    // Permitir números, fracciones, exponentes y operaciones básicas
    const validPattern = /^[0-9\-+*/()^.\/×÷]+$/;
    return validPattern.test(normalized) && normalized.length > 0;
  };

  // Generador de ejercicios mejorado y verificado
  const generateExercises = useCallback((level) => {
    const exercises = [];
    
    try {
      switch(level) {
        case 0: // Producto de potencias de igual base
          for(let i = 0; i < 15; i++) {
            const base = getRandomInt(2, 5);
            const exp1 = getRandomInt(1, 4);
            const exp2 = getRandomInt(1, 4);
            const answer = exp1 + exp2;
            
            // Verificación triple
            const check1 = exp1 + exp2;
            const check2 = parseInt(exp1) + parseInt(exp2);
            const check3 = Number(exp1) + Number(exp2);
            
            if (check1 === check2 && check2 === check3 && check1 === answer) {
              exercises.push({
                question: `${base}^${exp1} × ${base}^${exp2} = ${base}^?`,
                answer: answer.toString(),
                hint: `Suma los exponentes: ${exp1} + ${exp2} = ${answer}`,
                explanation: `Como las bases son iguales (${base}), sumamos los exponentes: ${exp1} + ${exp2} = ${answer}`,
                type: 'numeric'
              });
            }
          }
          break;

        case 1: // Cociente de potencias de igual base
          for(let i = 0; i < 15; i++) {
            const base = getRandomInt(2, 5);
            const exp1 = getRandomInt(4, 7);
            const exp2 = getRandomInt(1, 3);
            const answer = exp1 - exp2;
            
            // Verificación triple
            const check1 = exp1 - exp2;
            const check2 = parseInt(exp1) - parseInt(exp2);
            const check3 = Number(exp1) - Number(exp2);
            
            if (check1 === check2 && check2 === check3 && check1 === answer && answer > 0) {
              exercises.push({
                question: `${base}^${exp1} ÷ ${base}^${exp2} = ${base}^?`,
                answer: answer.toString(),
                hint: `Resta los exponentes: ${exp1} - ${exp2} = ${answer}`,
                explanation: `Como las bases son iguales (${base}), restamos los exponentes: ${exp1} - ${exp2} = ${answer}`,
                type: 'numeric'
              });
            }
          }
          break;

        case 2: // Potencia de una potencia
          for(let i = 0; i < 15; i++) {
            const base = getRandomInt(2, 4);
            const exp1 = getRandomInt(2, 4);
            const exp2 = getRandomInt(2, 3);
            const answer = exp1 * exp2;
            
            // Verificación triple
            const check1 = exp1 * exp2;
            const check2 = parseInt(exp1) * parseInt(exp2);
            const check3 = Number(exp1) * Number(exp2);
            
            if (check1 === check2 && check2 === check3 && check1 === answer) {
              exercises.push({
                question: `(${base}^${exp1})^${exp2} = ${base}^?`,
                answer: answer.toString(),
                hint: `Multiplica los exponentes: ${exp1} × ${exp2} = ${answer}`,
                explanation: `En potencia de potencia multiplicamos exponentes: ${exp1} × ${exp2} = ${answer}`,
                type: 'numeric'
              });
            }
          }
          break;

        case 3: // Potencia de un producto
          for(let i = 0; i < 15; i++) {
            const base1 = getRandomInt(2, 4);
            const base2 = getRandomInt(2, 4);
            const exp = getRandomInt(2, 4);
            
            // Verificación triple de cálculos
            const answer1_v1 = Math.pow(base1, exp);
            const answer1_v2 = base1 ** exp;
            let answer1_v3 = 1;
            for(let j = 0; j < exp; j++) answer1_v3 *= base1;
            
            const answer2_v1 = Math.pow(base2, exp);
            const answer2_v2 = base2 ** exp;
            let answer2_v3 = 1;
            for(let j = 0; j < exp; j++) answer2_v3 *= base2;
            
            if (answer1_v1 === answer1_v2 && answer1_v2 === answer1_v3 &&
                answer2_v1 === answer2_v2 && answer2_v2 === answer2_v3) {
              
              exercises.push({
                question: `(${base1} × ${base2})^${exp} = ? × ?`,
                answer: `${answer1_v1}×${answer2_v1}`,
                acceptedAnswers: [`${answer1_v1}×${answer2_v1}`, `${answer1_v1} × ${answer2_v1}`, `${answer1_v1}*${answer2_v1}`],
                hint: `Eleva cada factor: ${base1}^${exp} × ${base2}^${exp} = ${answer1_v1} × ${answer2_v1}`,
                explanation: `Distribuimos el exponente: ${base1}^${exp} = ${answer1_v1} y ${base2}^${exp} = ${answer2_v1}`,
                type: 'product'
              });
            }
          }
          break;

        case 4: // Potencia de un cociente
          for(let i = 0; i < 15; i++) {
            const num = getRandomInt(2, 4);
            const den = getRandomInt(2, 4);
            const exp = getRandomInt(2, 3);
            
            // Verificación triple
            const answerNum_v1 = Math.pow(num, exp);
            const answerNum_v2 = num ** exp;
            let answerNum_v3 = 1;
            for(let j = 0; j < exp; j++) answerNum_v3 *= num;
            
            const answerDen_v1 = Math.pow(den, exp);
            const answerDen_v2 = den ** exp;
            let answerDen_v3 = 1;
            for(let j = 0; j < exp; j++) answerDen_v3 *= den;
            
            if (answerNum_v1 === answerNum_v2 && answerNum_v2 === answerNum_v3 &&
                answerDen_v1 === answerDen_v2 && answerDen_v2 === answerDen_v3) {
              
              exercises.push({
                question: `(${num}/${den})^${exp} = ?/?`,
                answer: `${answerNum_v1}/${answerDen_v1}`,
                acceptedAnswers: [`${answerNum_v1}/${answerDen_v1}`, `${answerNum_v1} / ${answerDen_v1}`],
                hint: `Eleva numerador y denominador: ${num}^${exp}/${den}^${exp} = ${answerNum_v1}/${answerDen_v1}`,
                explanation: `Distribuimos el exponente: numerador ${num}^${exp} = ${answerNum_v1}, denominador ${den}^${exp} = ${answerDen_v1}`,
                type: 'fraction'
              });
            }
          }
          break;

        case 5: // Potencia con exponente cero
          for(let i = 0; i < 15; i++) {
            if(i < 10) {
              const base = getRandomInt(2, 20);
              exercises.push({
                question: `${base}^0 = ?`,
                answer: "1",
                hint: `Cualquier número diferente de cero elevado a 0 es 1`,
                explanation: `Por definición, ${base}^0 = 1 (ya que ${base} ≠ 0)`,
                type: 'numeric'
              });
            } else {
              const frac = getRandomFraction();
              exercises.push({
                question: `(${frac.num}/${frac.den})^0 = ?`,
                answer: "1",
                hint: `Cualquier número diferente de cero elevado a 0 es 1`,
                explanation: `Por definición, (${frac.num}/${frac.den})^0 = 1`,
                type: 'numeric'
              });
            }
          }
          break;

        case 6: // Potencia con exponente negativo
          for(let i = 0; i < 15; i++) {
            const base = getRandomInt(2, 5);
            const exp = getRandomInt(1, 3);
            
            // Verificación triple del denominador
            const denominator_v1 = Math.pow(base, exp);
            const denominator_v2 = base ** exp;
            let denominator_v3 = 1;
            for(let j = 0; j < exp; j++) denominator_v3 *= base;
            
            if (denominator_v1 === denominator_v2 && denominator_v2 === denominator_v3) {
              exercises.push({
                question: `${base}^(-${exp}) = ?`,
                answer: `1/${denominator_v1}`,
                acceptedAnswers: [`1/${denominator_v1}`, `1 / ${denominator_v1}`],
                hint: `${base}^(-${exp}) = 1/${base}^${exp} = 1/${denominator_v1}`,
                explanation: `Un exponente negativo invierte la fracción: ${base}^(-${exp}) = 1/${base}^${exp} = 1/${denominator_v1}`,
                type: 'fraction'
              });
            }
          }
          break;

        case 7: // Desafío final - combinado
          const properties = [0,1,2,3,4,5,6];
          for(let i = 0; i < 15; i++) {
            const randomProperty = properties[i % 7];
            const tempExercises = generateExercises(randomProperty);
            if (tempExercises && tempExercises.length > 0) {
              exercises.push(tempExercises[0]);
            }
          }
          break;
          
        default:
          throw new Error(`Invalid level: ${level}`);
      }
    } catch (error) {
      console.error('Error generating exercises:', error);
      return [];
    }

    return exercises.slice(0, 15);
  }, []);

  // Iniciar nivel
  const startLevel = useCallback((levelIndex) => {
    if (levelIndex < 0 || levelIndex >= levels.length) {
      console.error('Invalid level index:', levelIndex);
      return;
    }
    
    setIsLoading(true);
    try {
      setCurrentLevel(levelIndex);
      setCurrentExercise(0);
      setScore(0);
      setUserAnswer('');
      setShowFeedback(false);
      setHint('');
      
      const newExercises = generateExercises(levelIndex);
      if (newExercises.length === 0) {
        throw new Error('No exercises generated');
      }
      
      setExercises(newExercises);
      setGameState('tutorial');
    } catch (error) {
      console.error('Error starting level:', error);
      alert('Error al iniciar el nivel. Por favor, intenta de nuevo.');
    } finally {
      setIsLoading(false);
    }
  }, [generateExercises]);

  const startExercises = useCallback(() => {
    setGameState('playing');
    setUserAnswer('');
    setShowFeedback(false);
    setHint('');
  }, []);

  // Verificar respuesta mejorada
  const checkAnswer = useCallback(() => {
    if (!userAnswer.trim()) {
      alert('Por favor, ingresa una respuesta');
      return;
    }

    if (!validateInput(userAnswer)) {
      alert('Por favor, ingresa una respuesta válida (solo números, operadores matemáticos básicos)');
      return;
    }

    const currentEx = exercises[currentExercise];
    if (!currentEx) {
      console.error('No current exercise found');
      return;
    }

    const normalizedUserAnswer = normalizeAnswer(userAnswer);
    const normalizedCorrectAnswer = normalizeAnswer(currentEx.answer);
    
    let correct = false;
    
    // Verificar respuesta principal
    if (normalizedUserAnswer === normalizedCorrectAnswer) {
      correct = true;
    }
    
    // Verificar respuestas alternativas si existen
    if (!correct && currentEx.acceptedAnswers) {
      correct = currentEx.acceptedAnswers.some(answer => 
        normalizeAnswer(answer) === normalizedUserAnswer
      );
    }
    
    setIsCorrect(correct);
    setShowFeedback(true);
    
    if (correct) {
      setScore(prevScore => prevScore + 1);
    }
    
    setHint(currentEx.hint || '');
  }, [userAnswer, exercises, currentExercise]);

  const nextExercise = useCallback(() => {
    if (currentExercise < exercises.length - 1) {
      setCurrentExercise(prev => prev + 1);
      setUserAnswer('');
      setShowFeedback(false);
      setHint('');
    } else {
      // Nivel completado
      if (score >= 12) {
        setCompletedLevels(prev => {
          if (!prev.includes(currentLevel)) {
            return [...prev, currentLevel];
          }
          return prev;
        });
      }
      setGameState('results');
    }
  }, [currentExercise, exercises.length, score, currentLevel]);

  const getMotivationalMessage = useCallback(() => {
    if (score >= 14) return "¡Excelente! ¡Eres un maestro de las potencias!";
    if (score >= 12) return "¡Muy bien! Has dominado este nivel.";
    if (score >= 10) return "¡Buen trabajo! Sigue practicando.";
    return "¡No te rindas! La práctica hace al maestro.";
  }, [score]);

  const resetGame = useCallback(() => {
    setGameState('menu');
    setCurrentLevel(0);
    setCurrentExercise(0);
    setScore(0);
    setUserAnswer('');
    setShowFeedback(false);
    setHint('');
  }, []);

  // Manejo de teclas
  useEffect(() => {
    const handleKeyPress = (event) => {
      if (gameState === 'playing' && !showFeedback && event.key === 'Enter') {
        event.preventDefault();
        checkAnswer();
      }
    };

    window.addEventListener('keypress', handleKeyPress);
    return () => window.removeEventListener('keypress', handleKeyPress);
  }, [gameState, showFeedback, checkAnswer]);

  // Loading state
  if (isLoading) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-blue-500 to-purple-700 flex items-center justify-center">
        <div className="bg-white rounded-lg p-8 shadow-xl text-center">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
          <p className="text-gray-700">Cargando ejercicios...</p>
        </div>
      </div>
    );
  }

  // Componente del menú principal - Optimizado para móviles
  if (gameState === 'menu') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-blue-600 to-purple-700 p-2 sm:p-4">
        <div className="max-w-6xl mx-auto">
          <div className="text-center mb-6 sm:mb-8 px-4">
            <h1 className="text-2xl sm:text-3xl md:text-4xl font-bold text-white mb-2 sm:mb-4">
              🎯 Maestro de las Potencias
            </h1>
            <p className="text-base sm:text-lg md:text-xl text-blue-100">
              Domina todas las propiedades de potenciación en números racionales
            </p>
          </div>

          <div className="grid gap-3 sm:gap-4 md:grid-cols-2 lg:grid-cols-3">
            {levels.map((level, index) => (
              <div 
                key={index}
                className={`bg-white rounded-lg p-4 sm:p-6 shadow-lg cursor-pointer transition-all hover:shadow-xl hover:scale-105 active:scale-95 ${
                  completedLevels.includes(index) ? 'bg-green-50 border-2 border-green-500' : ''
                } touch-manipulation`}
                onClick={() => startLevel(index)}
                role="button"
                tabIndex={0}
                onKeyPress={(e) => e.key === 'Enter' && startLevel(index)}
                aria-label={`Iniciar nivel ${index + 1}: ${level.title}`}
              >
                <div className="flex items-center justify-between mb-3">
                  <h3 className="text-base sm:text-lg font-bold text-gray-800">
                    Nivel {index + 1}
                  </h3>
                  {completedLevels.includes(index) && (
                    <CheckCircle className="text-green-500 flex-shrink-0" size={20} />
                  )}
                </div>
                <h4 className="font-semibold text-purple-700 mb-2 text-sm sm:text-base">
                  {level.title}
                </h4>
                <p className="text-xs sm:text-sm text-gray-600 mb-3 leading-relaxed">
                  {level.description}
                </p>
                <div className="bg-gray-100 p-2 sm:p-3 rounded text-center font-mono text-purple-800 text-xs sm:text-sm break-all">
                  {level.property}
                </div>
              </div>
            ))}
          </div>

          <div className="text-center mt-6 sm:mt-8">
            <div className="bg-white/20 rounded-lg p-4 max-w-md mx-auto">
              <Trophy className="mx-auto text-yellow-300 mb-2" size={28} />
              <p className="text-white text-sm sm:text-base">
                Niveles completados: {completedLevels.length}/8
              </p>
              {completedLevels.length === 8 && (
                <p className="text-yellow-300 text-sm font-bold mt-2">
                  🏆 ¡Todos los niveles completados!
                </p>
              )}
            </div>
          </div>
        </div>
      </div>
    );
  }

  // Componente del tutorial - Optimizado para móviles
  if (gameState === 'tutorial') {
    const level = levels[currentLevel];
    return (
      <div className={`min-h-screen bg-gradient-to-br ${level.color} p-2 sm:p-4`}>
        <div className="max-w-4xl mx-auto bg-white rounded-lg shadow-xl p-4 sm:p-6 md:p-8">
          {/* Header con botón de regreso */}
          <div className="flex items-center justify-between mb-4 sm:mb-6">
            <button
              onClick={resetGame}
              className="flex items-center text-gray-600 hover:text-gray-800 transition-colors"
              aria-label="Volver al menú principal"
            >
              <Home className="mr-2" size={20} />
              <span className="hidden sm:inline">Menú</span>
            </button>
            <span className="text-sm sm:text-base text-gray-600 font-medium">
              Nivel {currentLevel + 1} de 8
            </span>
          </div>

          <div className="text-center mb-4 sm:mb-6">
            <BookOpen className="mx-auto text-green-600 mb-3 sm:mb-4" size={40} />
            <h2 className="text-xl sm:text-2xl md:text-3xl font-bold text-gray-800 mb-2 sm:mb-4">
              {level.title}
            </h2>
            <div className="bg-blue-100 p-3 sm:p-4 rounded-lg mb-4">
              <p className="text-base sm:text-lg font-mono text-blue-800 break-all">
                {level.property}
              </p>
            </div>
          </div>

          <div className="mb-4 sm:mb-6">
            <p className="text-gray-700 mb-4 text-sm sm:text-base leading-relaxed">
              {level.description}
            </p>
            
            <div className="bg-yellow-50 p-3 sm:p-4 rounded-lg">
              <h4 className="font-bold text-yellow-800 mb-3 flex items-center text-sm sm:text-base">
                <Lightbulb className="mr-2 flex-shrink-0" size={18} />
                Consejos importantes:
              </h4>
              <ul className="space-y-2">
                {level.tips.map((tip, index) => (
                  <li key={index} className="flex items-start text-yellow-800 text-xs sm:text-sm">
                    <Star className="mr-2 mt-1 flex-shrink-0" size={14} />
                    <span>{tip}</span>
                  </li>
                ))}
              </ul>
            </div>
          </div>

          <div className="bg-gray-50 p-3 sm:p-4 rounded-lg mb-4 sm:mb-6">
            <h4 className="font-bold text-gray-800 mb-2 text-sm sm:text-base">
              📋 Instrucciones del nivel:
            </h4>
            <ul className="text-xs sm:text-sm text-gray-700 space-y-1 leading-relaxed">
              <li>• Responderás 15 ejercicios sobre esta propiedad</li>
              <li>• Necesitas al menos 12 respuestas correctas para aprobar</li>
              <li>• Escribe tu respuesta en el formato exacto que se pide</li>
              <li>• ¡Puedes solicitar pistas si lo necesitas!</li>
            </ul>
          </div>

          <div className="text-center">
            <button 
              onClick={startExercises}
              className="bg-green-600 text-white px-6 sm:px-8 py-3 rounded-lg font-bold hover:bg-green-700 active:bg-green-800 transition-colors flex items-center mx-auto text-sm sm:text-base touch-manipulation"
              aria-label="Comenzar ejercicios del nivel"
            >
              ¡Comenzar Ejercicios! 
              <ChevronRight className="ml-2" size={18} />
            </button>
          </div>
        </div>
      </div>
    );
  }

  // Componente de ejercicios - Completamente optimizado para móviles
  if (gameState === 'playing') {
    const currentEx = exercises[currentExercise];
    if (!currentEx) {
      return (
        <div className="min-h-screen bg-red-500 flex items-center justify-center p-4">
          <div className="bg-white rounded-lg p-6 text-center">
            <p className="text-red-600 font-bold">Error: No se encontró el ejercicio</p>
            <button onClick={resetGame} className="mt-4 bg-blue-600 text-white px-4 py-2 rounded">
              Volver al Menú
            </button>
          </div>
        </div>
      );
    }
    
    return (
      <div className={`min-h-screen bg-gradient-to-br ${levels[currentLevel].color} p-2 sm:p-4`}>
        <div className="max-w-4xl mx-auto">
          {/* Header con progreso - Optimizado para móviles */}
          <div className="bg-white rounded-lg p-3 sm:p-4 mb-4 sm:mb-6 shadow-lg">
            <div className="flex items-center justify-between mb-2">
              <button
                onClick={resetGame}
                className="flex items-center text-gray-600 hover:text-gray-800 transition-colors"
                aria-label="Volver al menú principal"
              >
                <Home size={16} />
                <span className="ml-1 text-xs sm:text-sm">Menú</span>
              </button>
              <span className="text-xs sm:text-sm font-semibold text-purple-600">
                {currentExercise + 1}/15
              </span>
            </div>
            <div className="mb-2">
              <h3 className="text-xs sm:text-sm font-semibold text-gray-600 truncate">
                Nivel {currentLevel + 1}: {levels[currentLevel].title}
              </h3>
            </div>
            <div className="w-full bg-gray-200 rounded-full h-2 mb-2">
              <div 
                className="bg-purple-600 h-2 rounded-full transition-all duration-300"
                style={{ width: `${((currentExercise + 1) / 15) * 100}%` }}
                role="progressbar"
                aria-valuenow={currentExercise + 1}
                aria-valuemax={15}
              ></div>
            </div>
            <div className="text-center">
              <span className="text-xs sm:text-sm text-gray-600">
                ✅ {score}/15 (Necesitas 12 para aprobar)
              </span>
            </div>
          </div>

          {/* Ejercicio */}
          <div className="bg-white rounded-lg p-4 sm:p-6 md:p-8 shadow-xl">
            <div className="text-center mb-4 sm:mb-6">
              <h3 className="text-lg sm:text-xl md:text-2xl font-bold text-gray-800 mb-3 sm:mb-4">
                Resuelve el siguiente ejercicio:
              </h3>
              <div className="bg-blue-50 p-4 sm:p-6 rounded-lg">
                <p className="text-lg sm:text-xl md:text-2xl font-mono text-blue-800 break-all">
                  {currentEx.question}
                </p>
              </div>
            </div>

            {!showFeedback ? (
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-2">
                    Tu respuesta:
                  </label>
                  <input
                    type="text"
                    value={userAnswer}
                    onChange={(e) => setUserAnswer(e.target.value)}
                    className="w-full p-3 sm:p-4 border-2 border-gray-300 rounded-lg font-mono text-center text-base sm:text-lg focus:border-purple-500 focus:outline-none touch-manipulation"
                    placeholder="Escribe tu respuesta aquí..."
                    onKeyPress={(e) => e.key === 'Enter' && checkAnswer()}
                    autoComplete="off"
                    autoCapitalize="none"
                    autoCorrect="off"
                    spellCheck="false"
                    aria-label="Campo de respuesta"
                  />
                </div>
                <div className="flex flex-col sm:flex-row gap-3 justify-center">
                  <button 
                    onClick={checkAnswer}
                    disabled={!userAnswer.trim()}
                    className="bg-purple-600 text-white px-6 py-3 rounded-lg font-bold hover:bg-purple-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors touch-manipulation text-sm sm:text-base"
                    aria-label="Verificar respuesta"
                  >
                    ✓ Verificar Respuesta
                  </button>
                  <button 
                    onClick={() => setHint(currentEx.hint)}
                    className="bg-yellow-500 text-white px-6 py-3 rounded-lg font-bold hover:bg-yellow-600 transition-colors touch-manipulation text-sm sm:text-base"
                    aria-label="Mostrar pista"
                  >
                    💡 Pista
                  </button>
                </div>
              </div>
            ) : (
              <div className={`text-center p-4 sm:p-6 rounded-lg ${
                isCorrect ? 'bg-green-50 border-2 border-green-500' : 'bg-red-50 border-2 border-red-500'
              }`}>
                <div className="flex items-center justify-center mb-4">
                  {isCorrect ? (
                    <CheckCircle className="text-green-600 mr-3" size={28} />
                  ) : (
                    <XCircle className="text-red-600 mr-3" size={28} />
                  )}
                  <h4 className={`text-lg sm:text-xl font-bold ${
                    isCorrect ? 'text-green-800' : 'text-red-800'
                  }`}>
                    {isCorrect ? '¡Correcto!' : '¡Incorrecto!'}
                  </h4>
                </div>
                
                <div className="mb-4">
                  <p className="text-base sm:text-lg mb-2">
                    <strong>Tu respuesta:</strong> {userAnswer}
                  </p>
                  <p className="text-base sm:text-lg mb-4">
                    <strong>Respuesta correcta:</strong> {currentEx.answer}
                  </p>
                </div>
                
                <div className="bg-blue-50 p-3 sm:p-4 rounded-lg mb-4">
                  <p className="text-xs sm:text-sm text-blue-800 leading-relaxed">
                    <strong>Explicación:</strong> {currentEx.explanation}
                  </p>
                </div>

                <button 
                  onClick={nextExercise}
                  className="bg-blue-600 text-white px-6 py-3 rounded-lg font-bold hover:bg-blue-700 transition-colors touch-manipulation text-sm sm:text-base"
                  aria-label={currentExercise < 14 ? 'Ir al siguiente ejercicio' : 'Ver resultados del nivel'}
                >
                  {currentExercise < 14 ? 'Siguiente Ejercicio →' : 'Ver Resultados 🏁'}
                </button>
              </div>
            )}

            {hint && (
              <div className="mt-4 bg-yellow-50 border-l-4 border-yellow-400 p-3 sm:p-4 rounded">
                <p className="text-yellow-800 text-xs sm:text-sm leading-relaxed">
                  <strong>💡 Pista:</strong> {hint}
                </p>
              </div>
            )}
          </div>
        </div>
      </div>
    );
  }

  // Componente de resultados - Optimizado para móviles
  if (gameState === 'results') {
    const passed = score >= 12;
    const percentage = Math.round((score / 15) * 100);
    
    return (
      <div className="min-h-screen bg-gradient-to-br from-orange-500 to-red-600 p-2 sm:p-4">
        <div className="max-w-4xl mx-auto bg-white rounded-lg shadow-xl p-4 sm:p-6 md:p-8 text-center">
          <div className="mb-4 sm:mb-6">
            {passed ? (
              <Trophy className="mx-auto text-yellow-500 mb-4" size={48} />
            ) : (
              <XCircle className="mx-auto text-red-500 mb-4" size={48} />
            )}
            
            <h2 className={`text-2xl sm:text-3xl font-bold mb-2 ${
              passed ? 'text-green-700' : 'text-red-700'
            }`}>
              {passed ? '🎉 ¡Nivel Completado!' : '💪 ¡Sigue Intentando!'}
            </h2>
            
            <p className="text-lg sm:text-xl text-gray-700">
              {getMotivationalMessage()}
            </p>
          </div>

          <div className="bg-gray-50 p-4 sm:p-6 rounded-lg mb-4 sm:mb-6">
            <h3 className="text-lg sm:text-xl font-bold text-gray-800 mb-4">
              Resultados del Nivel {currentLevel + 1}
            </h3>
            <div className="grid grid-cols-2 gap-4 text-center mb-4">
              <div className="bg-white p-3 sm:p-4 rounded-lg">
                <p className="text-xl sm:text-2xl font-bold text-green-600">{score}</p>
                <p className="text-xs sm:text-sm text-gray-600">Correctas</p>
              </div>
              <div className="bg-white p-3 sm:p-4 rounded-lg">
                <p className="text-xl sm:text-2xl font-bold text-red-600">{15 - score}</p>
                <p className="text-xs sm:text-sm text-gray-600">Incorrectas</p>
              </div>
            </div>
            
            <div className="mb-4">
              <div className="w-full bg-gray-200 rounded-full h-3 sm:h-4">
                <div 
                  className={`h-3 sm:h-4 rounded-full transition-all duration-500 ${
                    passed ? 'bg-green-500' : 'bg-red-500'
                  }`}
                  style={{ width: `${percentage}%` }}
                  role="progressbar"
                  aria-valuenow={percentage}
                  aria-valuemax={100}
                ></div>
              </div>
              <p className="text-sm text-gray-600 mt-2">
                {percentage}% de precisión {passed ? '(¡Aprobado!)' : '(Necesitas 80% para aprobar)'}
              </p>
            </div>
          </div>

          <div className="flex flex-col sm:flex-row gap-3 justify-center mb-4">
            <button 
              onClick={resetGame}
              className="bg-blue-600 text-white px-4 sm:px-6 py-3 rounded-lg font-bold hover:bg-blue-700 transition-colors touch-manipulation text-sm sm:text-base"
              aria-label="Volver al menú principal"
            >
              🏠 Volver al Menú
            </button>
            
            {!passed && (
              <button 
                onClick={() => startLevel(currentLevel)}
                className="bg-orange-600 text-white px-4 sm:px-6 py-3 rounded-lg font-bold hover:bg-orange-700 transition-colors touch-manipulation text-sm sm:text-base"
                aria-label="Reintentar el nivel actual"
              >
                🔄 Reintentar Nivel
              </button>
            )}
            
            {passed && currentLevel < 7 && (
              <button 
                onClick={() => startLevel(currentLevel + 1)}
                className="bg-green-600 text-white px-4 sm:px-6 py-3 rounded-lg font-bold hover:bg-green-700 transition-colors touch-manipulation text-sm sm:text-base"
                aria-label="Ir al siguiente nivel"
              >
                ➡️ Siguiente Nivel
              </button>
            )}
          </div>

          {passed && currentLevel === 7 && completedLevels.length === 8 && (
            <div className="bg-yellow-50 border-2 border-yellow-400 p-4 rounded-lg">
              <h4 className="text-lg sm:text-xl font-bold text-yellow-800 mb-2">
                🎖️ ¡Felicidades!
              </h4>
              <p className="text-yellow-700 text-sm sm:text-base leading-relaxed">
                Has completado todos los niveles del juego. ¡Eres oficialmente un <strong>Maestro de las Propiedades de Potenciación</strong>!
              </p>
            </div>
          )}
        </div>
      </div>
    );
  }

  return null;
};

export default PotenciacionGame;
