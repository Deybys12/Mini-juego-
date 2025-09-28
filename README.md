import React, { useState, useEffect } from 'react';
import { Check, X, Trophy, Zap, Clock, Heart, Users, MessageSquare } from 'lucide-react';

// Mezcla un array usando Fisher-Yates
function shuffleArray(array) {
  const newArray = [...array]; // Copia para no mutar el original
  for (let i = newArray.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
  }
  return newArray;
}

// Mezcla opciones y devuelve el nuevo índice correcto
function shuffleOptions(options, correctIndex) {
  const indexed = options.map((opt, idx) => ({ text: opt, isCorrect: idx === correctIndex }));
  for (let i = indexed.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [indexed[i], indexed[j]] = [indexed[j], indexed[i]];
  }
  return {
    options: indexed.map(o => o.text),
    correct: indexed.findIndex(o => o.isCorrect)
  };
}

const App = () => {
  const [gameState, setGameState] = useState('welcome');
  const [playerName, setPlayerName] = useState('');
  const [difficulty, setDifficulty] = useState('medio');
  const [totalQuestions, setTotalQuestions] = useState(12);
  const [currentQuestion, setCurrentQuestion] = useState(0);
  const [score, setScore] = useState(0);
  const [streak, setStreak] = useState(0);
  const [lives, setLives] = useState(3);
  const [selectedAnswer, setSelectedAnswer] = useState(null);
  const [isCorrect, setIsCorrect] = useState(null);
  const [showExplanation, setShowExplanation] = useState(false);
  const [timeLeft, setTimeLeft] = useState(30);
  const [badges, setBadges] = useState({
    liderTransformacional: false,
    comunicadorClaro: false,
    equipoMaestro: false,
    maestroCNP: false,
    competente: false
  });
  const [answers, setAnswers] = useState([]);
  const [shuffledQuestions, setShuffledQuestions] = useState([]);

  // Todas las preguntas para los 3 niveles (básico, medio, difícil) - reformuladas con opciones trampa
  const questions = {
    // ... (tu objeto questions aquí, sin cambios)
    basico: [
      {
        id: 1,
        question: "Según Koontz, ¿cuál es la esencia fundamental del liderazgo?",
        options: [
          "Mantener el control jerárquico sobre los subordinados",
          "Influir en otras personas para que trabajen voluntariamente hacia los objetivos organizacionales",
          "Garantizar que todas las órdenes sean obedecidas sin cuestionamiento",
          "Supervisar constantemente el desempeño individual de cada empleado"
        ],
        correct: 1,
        explanation: {
          correct: "El liderazgo es el proceso de influir en otras personas para que trabajen de manera voluntaria y con entusiasmo hacia los objetivos de la organización, no se trata solo de autoridad o control.",
          incorrect: [
            "El control jerárquico es más característico de la autoridad que del liderazgo.",
            "Las órdenes sin cuestionamiento no fomentan el compromiso voluntario.",
            "La supervisión constante no inspira entusiasmo ni trabajo voluntario."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 2,
        question: "¿Qué diferencia fundamental existe entre un grupo y un equipo de trabajo?",
        options: [
          "Los equipos tienen más miembros que los grupos",
          "Los grupos se reúnen con más frecuencia que los equipos",
          "Los equipos comparten responsabilidad y tienen un propósito común claro",
          "Los grupos son más formales en su estructura organizacional"
        ],
        correct: 2,
        explanation: {
          correct: "La diferencia principal es que un equipo comparte responsabilidad, confía en las capacidades de los demás y se coordina con un propósito claro, mientras que un grupo es simplemente un conjunto de personas.",
          incorrect: [
            "El tamaño no define la diferencia entre grupo y equipo.",
            "La frecuencia de reuniones no es la característica distintiva.",
            "La formalidad no es lo que diferencia a un grupo de un equipo."
          ]
        },
        category: 'equipos'
      },
      {
        id: 3,
        question: "En el contexto de resolución de conflictos, ¿por qué la escucha activa es tan efectiva?",
        options: [
          "Permite interrumpir al otro para corregir sus errores inmediatamente",
          "Facilita la comprensión de las perspectivas de todas las partes involucradas",
          "Ayuda a ganar tiempo mientras se piensa en una respuesta defensiva",
          "Demuestra superioridad intelectual sobre la otra persona"
        ],
        correct: 1,
        explanation: {
          correct: "La escucha activa permite comprender las perspectivas de todas las partes involucradas y encontrar soluciones efectivas.",
          incorrect: [
            "Interrumpir impide la comprensión efectiva.",
            "Ganar tiempo no es el propósito de la escucha activa.",
            "La escucha activa no busca demostrar superioridad."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 4,
        question: "¿Cuál es el componente más crítico para establecer un equipo efectivo desde el inicio?",
        options: [
          "Contratar a los profesionales más calificados del mercado",
          "Implementar tecnología de última generación para la colaboración",
          "Definir metas claras y comunes que todos los miembros comprendan",
          "Establecer reglas estrictas de comportamiento y puntualidad"
        ],
        correct: 2,
        explanation: {
          correct: "Las metas claras y comunes proporcionan dirección y propósito común para todos los miembros del equipo.",
          incorrect: [
            "Los profesionales calificados son importantes, pero sin metas claras no hay dirección.",
            "La tecnología facilita la colaboración, pero no es el fundamento del equipo.",
            "Las reglas son necesarias, pero no son el componente más crítico inicial."
          ]
        },
        category: 'equipos'
      },
      {
        id: 5,
        question: "¿Qué caracteriza fundamentalmente al liderazgo ético en una organización?",
        options: [
          "Tomar decisiones rápidas sin consultar a otros para mantener la eficiencia",
          "Priorizar siempre los resultados financieros sobre consideraciones personales",
          "Modelar consistentemente los comportamientos y valores que se esperan de otros",
          "Evitar la responsabilidad personal delegando todas las decisiones importantes"
        ],
        correct: 2,
        explanation: {
          correct: "Los líderes éticos deben servir como ejemplo de los valores y comportamientos que desean ver en sus seguidores.",
          incorrect: [
            "Tomar decisiones sin consultar puede ignorar perspectivas importantes.",
            "Priorizar solo resultados financieros no es ético.",
            "Evitar la responsabilidad no es característico del liderazgo ético."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 6,
        question: "Durante una crisis organizacional, ¿cuál enfoque comunicacional es más efectivo?",
        options: [
          "Retener información crítica hasta tener todos los detalles completos",
          "Comunicar solo a través de canales informales para evitar pánico",
          "Mantener comunicación clara, frecuente y transparente con todas las partes",
          "Limitar la comunicación a los niveles jerárquicos superiores únicamente"
        ],
        correct: 2,
        explanation: {
          correct: "La comunicación clara, frecuente y transparente reduce la incertidumbre y mantiene a todos informados durante una crisis.",
          incorrect: [
            "Retener información genera rumores y ansiedad.",
            "La comunicación informal puede ser inexacta en crisis.",
            "Limitar la comunicación impide la coordinación efectiva."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 7,
        question: "¿Cuál es el propósito principal de una reunión de equipo verdaderamente efectiva?",
        options: [
          "Evaluar individualmente el desempeño de cada miembro del equipo",
          "Informar unilateralmente sobre decisiones ya tomadas por la gerencia",
          "Compartir información, resolver problemas colectivos y alinear objetivos",
          "Reducir la comunicación informal entre los miembros del equipo"
        ],
        correct: 2,
        explanation: {
          correct: "Las reuniones efectivas sirven para compartir información, resolver problemas y alinear objetivos entre los miembros del equipo.",
          incorrect: [
            "La evaluación individual no es el propósito principal de reuniones de equipo.",
            "Informar unilateralmente no fomenta la colaboración.",
            "Reducir la comunicación informal puede afectar negativamente la cohesión."
          ]
        },
        category: 'equipos'
      },
      {
        id: 8,
        question: "¿Qué distingue al liderazgo transformacional de otros estilos de liderazgo?",
        options: [
          "El enfoque en el control estricto de todos los procesos operativos",
          "La capacidad de motivar a seguidores a trascender sus intereses personales",
          "La delegación total de responsabilidades sin supervisión",
          "El mantenimiento riguroso del status quo organizacional"
        ],
        correct: 1,
        explanation: {
          correct: "El liderazgo transformacional se centra en inspirar y motivar a los seguidores para que trasciendan sus intereses personales y se comprometan con la visión del líder.",
          incorrect: [
            "El control estricto es más característico del liderazgo transaccional.",
            "La delegación total sin supervisión no es liderazgo efectivo.",
            "Mantener el status quo no promueve el cambio transformacional."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 9,
        question: "¿Cuál es el beneficio más significativo del trabajo en equipo multidisciplinario?",
        options: [
          "Aumentar la carga de trabajo individual para mejorar la productividad",
          "Reducir la responsabilidad personal al distribuirla entre muchos",
          "Combinar habilidades complementarias para lograr objetivos comunes más efectivamente",
          "Evitar la toma de decisiones difíciles mediante el consenso automático"
        ],
        correct: 2,
        explanation: {
          correct: "El trabajo en equipo permite combinar habilidades complementarias de diferentes personas para lograr objetivos comunes de manera más efectiva.",
          incorrect: [
            "El trabajo en equipo debería optimizar, no aumentar la carga individual.",
            "El trabajo en equipo implica más responsabilidad compartida, no menos.",
            "El consenso automático no es realista ni deseable en equipos efectivos."
          ]
        },
        category: 'equipos'
      },
      {
        id: 10,
        question: "¿Por qué el modelo SBI (Situación-Comportamiento-Impacto) es efectivo para dar retroalimentación?",
        options: [
          "Permite criticar públicamente para que todos aprendan de los errores",
          "Se centra exclusivamente en los errores para evitar su repetición",
          "Proporciona un marco estructurado y específico para la retroalimentación constructiva",
          "Evita dar retroalimentación negativa para no herir sentimientos"
        ],
        correct: 2,
        explanation: {
          correct: "El modelo SBI proporciona un marco estructurado para dar retroalimentación específica y objetiva.",
          incorrect: [
            "Criticar en público puede ser humillante y dañar la confianza.",
            "Centrarse solo en errores ignora los logros y puede desmotivar.",
            "Evitar la retroalimentación impide el crecimiento y desarrollo."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 11,
        question: "¿Cuál comportamiento demuestra un liderazgo efectivo en la práctica diaria?",
        options: [
          "Fomentar competencia interna para motivar el alto desempeño",
          "Tomar decisiones importantes sin consultar para demostrar autoridad",
          "Evitar la comunicación directa para mantener distancia profesional",
          "Mantener informados a los empleados sobre decisiones que les afectan"
        ],
        correct: 3,
        explanation: {
          correct: "Mantener a los empleados informados sobre decisiones importantes fomenta la transparencia y la confianza en el liderazgo.",
          incorrect: [
            "Fomentar la competencia interna puede generar conflictos y disminuir la colaboración.",
            "Tomar decisiones sin consultar puede generar desconfianza.",
            "Evitar la comunicación directa socava la relación de confianza."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 12,
        question: "En el contexto de Cervecería Nacional, ¿qué representa el programa 'Transforma'?",
        options: [
          "Una iniciativa exclusiva para aumentar las ventas de productos",
          "Un programa de reducción de costos operativos en toda la organización",
          "Una estrategia de competencia directa con otras cervecerías",
          "Un programa para mejorar la calidad de vida de empleados y crear ambiente laboral positivo"
        ],
        correct: 3,
        explanation: {
          correct: "El programa 'Transforma' está diseñado para mejorar la calidad de vida de los empleados y crear un ambiente laboral positivo, lo que es parte del liderazgo transformacional de la empresa.",
          incorrect: [
            "El programa no tiene como objetivo principal aumentar ventas.",
            "El programa no tiene como objetivo principal reducir costos.",
            "El programa no tiene como objetivo principal competir con otras empresas."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 13,
        question: "¿Cómo se manifiesta el liderazgo con propósito en Cervecería Nacional?",
        options: [
          "Priorizando exclusivamente los beneficios económicos trimestrales",
          "Inspirando a las personas a dar lo mejor de sí con visión, valores y bienestar",
          "Evitando responsabilidades sociales para enfocarse en la producción",
          "Focalizándose únicamente en la competencia del mercado cervecero"
        ],
        correct: 1,
        explanation: {
          correct: "Cervecería Nacional refleja liderazgo con propósito al inspirar a las personas a dar lo mejor de sí con visión, valores y bienestar, como se ve en su compromiso con la sostenibilidad, el bienestar de los colaboradores y el programa 'Transforma'.",
          incorrect: [
            "La empresa no prioriza solo beneficios económicos.",
            "Esta es la respuesta correcta.",
            "La empresa no evita responsabilidades sociales.",
            "La empresa no se focaliza solo en competencia."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 14,
        question: "¿Qué tipo de equipos predominan en la estructura de Cervecería Nacional?",
        options: [
          "Equipos homogéneos con miembros de la misma área funcional",
          "Equipos temporales que se disuelven después de cada proyecto",
          "Equipos multidisciplinarios que combinan distintas áreas y conocimientos",
          "Equipos individuales que trabajan de forma aislada sin colaboración"
        ],
        correct: 2,
        explanation: {
          correct: "En Cervecería Nacional, los equipos de trabajo son multidisciplinarios: combinan gente de distintas áreas y conocimientos, lo que enriquece la toma de decisiones, mejora la creatividad y fortalece el compromiso con los valores de la empresa.",
          incorrect: [
            "Equipos homogéneos limitan la diversidad de perspectivas.",
            "Los equipos no son temporales, sino parte de la estructura organizacional.",
            "La empresa fomenta la colaboración, no el trabajo individual."
          ]
        },
        category: 'equipos'
      },
      {
        id: 15,
        question: "¿Cuál es un ejemplo concreto de comité interno en Cervecería Nacional?",
        options: [
          "Comité de Maximización de Beneficios Financieros",
          "Comité de Reducción de Personal Estratégico",
          "Comité de Competencia con Otras Marcas",
          "Comité de Sostenibilidad y Comité de Bienestar Laboral"
        ],
        correct: 3,
        explanation: {
          correct: "Cervecería Nacional tiene comités formales como el Comité de Sostenibilidad (que garantiza actuar de forma responsable con el medio ambiente) y el Comité de Bienestar Laboral (que se centra en mejorar la calidad de vida de los empleados).",
          incorrect: [
            "No se menciona un comité de maximización de beneficios.",
            "No se menciona un comité de reducción de personal.",
            "No se menciona un comité de competencia con otras marcas.",
            "Esta es la respuesta correcta."
          ]
        },
        category: 'equipos'
      }
    ],
    medio: [
      {
        id: 1,
        question: "¿Cómo demuestra Rodrigo Monteiro, presidente de Cervecería Nacional, su liderazgo transformacional en la práctica?",
        options: [
          "Supervisando estrictamente todas las operaciones diarias de la empresa",
          "Promoviendo el bienestar de colaboradores y fomentando la innovación",
          "Evitando la toma de decisiones estratégicas para delegar responsabilidad",
          "Manteniendo el mismo enfoque de gestión sin importar las circunstancias"
        ],
        correct: 1,
        explanation: {
          correct: "Rodrigo Monteiro practica un liderazgo transformacional que busca motivar a los colaboradores a ir más allá de sus intereses individuales para alcanzar un bien común. No se limita a supervisar, sino que promueve el bienestar de los colaboradores, fomenta la innovación y refuerza la responsabilidad social.",
          incorrect: [
            "La supervisión estricta no es liderazgo transformacional.",
            "Evitar la toma de decisiones no es liderazgo transformacional.",
            "Mantener el mismo enfoque no es adaptativo ni transformacional."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 2,
        question: "¿Qué combinación de estilos de liderazgo aplica Cervecería Nacional según su práctica organizacional?",
        options: [
          "Exclusivamente liderazgo transformacional en todas las situaciones",
          "Transformacional, situacional y participativo según las circunstancias",
          "Solo liderazgo autoritario para mantener el control operativo",
          "Exclusivamente liderazgo laissez-faire para fomentar la autonomía"
        ],
        correct: 1,
        explanation: {
          correct: "Cervecería Nacional combina liderazgo transformacional (inspirar y motivar), situacional (adaptar el estilo a las circunstancias, como durante la pandemia) y participativo (involucrar al equipo en decisiones importantes).",
          incorrect: [
            "La empresa combina varios estilos, no solo uno.",
            "Esta es la respuesta correcta.",
            "La empresa no se limita a un estilo autoritario.",
            "La empresa no se limita a un estilo laissez-faire."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 3,
        question: "Durante la pandemia, ¿cómo demostró Cervecería Nacional su capacidad de liderazgo situacional?",
        options: [
          "Manteniendo exactamente las mismas políticas y procedimientos que antes
