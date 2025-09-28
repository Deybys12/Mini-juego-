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
          "Manteniendo exactamente las mismas políticas y procedimientos que antes",
          "Ajustando su forma de trabajar y tomar decisiones para proteger empleados y consumidores",
          "Delegando completamente la gestión de la crisis a consultores externos",
          "Suspensión total de todas las operaciones hasta que pasara la crisis"
        ],
        correct: 1,
        explanation: {
          correct: "Durante la pandemia, la empresa ajustó su forma de trabajar y de tomar decisiones para proteger tanto a empleados como a consumidores, mostrando liderazgo situacional adaptado a las circunstancias.",
          incorrect: [
            "Mantener las mismas políticas no es adaptativo.",
            "Esta es la respuesta correcta.",
            "Delegar completamente no es liderazgo efectivo.",
            "Suspender operaciones no es una estrategia viable."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 4,
        question: "¿Cómo contribuyen los comités internos como el de Sostenibilidad a la gestión de Cervecería Nacional?",
        options: [
          "Creando conflictos entre departamentos por diferencias de opinión",
          "Limitando la comunicación entre empleados para mantener el enfoque",
          "Aumentando la burocracia sin aportar valor real a la organización",
          "Resolviendo problemas de manera colectiva y fomentando participación activa"
        ],
        correct: 3,
        explanation: {
          correct: "Los comités como el Comité de Sostenibilidad y el Comité de Bienestar Laboral cumplen funciones importantes como resolver problemas de manera colectiva, proponer mejoras internas y fomentar la participación activa de los colaboradores en la gestión de la empresa.",
          incorrect: [
            "Los comités no crean conflictos, sino soluciones.",
            "Los comités fomentan la comunicación, no la limitan.",
            "Los comités aportan valor a la empresa."
          ]
        },
        category: 'equipos'
      },
      {
        id: 5,
        question: "¿Qué técnicas específicas utiliza Cervecería Nacional para la toma de decisiones en grupo?",
        options: [
          "Decisiones unilaterales del presidente sin consulta previa",
          "Votación secreta exclusiva para todas las decisiones importantes",
          "Lluvia de ideas y grupo nominal con participación de múltiples partes",
          "Decisión por sorteo para garantizar imparcialidad absoluta"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional utiliza técnicas como lluvia de ideas (generar ideas sin críticas iniciales) y el grupo nominal (estructurado, donde cada persona aporta ideas, luego se discuten y se priorizan) para tomar decisiones en grupo.",
          incorrect: [
            "La empresa no toma decisiones unilaterales.",
            "La votación secreta no es la técnica principal.",
            "La decisión por sorteo no es una técnica profesional."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 6,
        question: "¿Cómo se estructura la comunicación organizacional en Cervecería Nacional?",
        options: [
          "Exclusivamente vertical, de arriba hacia abajo en la jerarquía",
          "Solo horizontal entre departamentos del mismo nivel",
          "Únicamente externa, enfocada en clientes y medios de comunicación",
          "Interna entre áreas y empleados, y externa con clientes y sociedad"
        ],
        correct: 3,
        explanation: {
          correct: "La comunicación en Cervecería Nacional se divide en interna (entre áreas, departamentos y empleados) y externa (con clientes, medios de comunicación y la sociedad).",
          incorrect: [
            "La comunicación no es exclusivamente vertical.",
            "La comunicación no es solo horizontal.",
            "La comunicación no es únicamente externa."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 7,
        question: "¿Qué herramientas utiliza Cervecería Nacional específicamente para su comunicación comercial?",
        options: [
          "Solo publicidad tradicional en medios masivos",
          "Exclusivamente redes sociales para llegar a consumidores",
          "Plataformas como Bees para comunicarse con tenderos y distribuidores",
          "Solo correos electrónicos para comunicación interna"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional utiliza plataformas como Bees para comunicarse directamente con tenderos y distribuidores, facilitando la relación comercial.",
          incorrect: [
            "La empresa no se limita a publicidad tradicional.",
            "La empresa no se limita a redes sociales.",
            "La comunicación comercial no es solo interna."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 8,
        question: "¿Cómo enfrenta Cervecería Nacional las barreras de comunicación como los 'filtros personales'?",
        options: [
          "Ignorando las resistencias al cambio como parte del proceso natural",
          "Aumentando la burocracia para controlar mejor la información",
          "Implementando talleres de liderazgo y programas de motivación",
          "Limitando la comunicación interna para evitar malentendidos"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional enfrenta barreras como filtros personales (resistencia al cambio) con estrategias activas como talleres de liderazgo y programas de motivación.",
          incorrect: [
            "La empresa no ignora las barreras.",
            "La empresa no aumenta la burocracia.",
            "La empresa no limita la comunicación interna."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 9,
        question: "¿Qué resultados concretos demuestran la efectividad del liderazgo en Cervecería Nacional?",
        options: [
          "Reducción significativa del número de empleados en la organización",
          "Cierre de operaciones en mercados internacionales no rentables",
          "Posición entre las top 5 de Merco Empresas 2024 y aporte al PIB de Panamá",
          "Aumento exclusivo de beneficios financieros sin considerar impacto social"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional está entre las top 5 de Merco Empresas 2024, un ranking de reputación corporativa, aporta un 1.6% al PIB de Panamá y genera un 5.7% de los empleos del sector, demostrando la efectividad de su liderazgo.",
          incorrect: [
            "La reducción de empleados no es un resultado positivo.",
            "El cierre de operaciones no es un resultado positivo.",
            "El aumento exclusivo de beneficios no refleja liderazgo integral."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 10,
        question: "¿Cómo demuestra Cervecería Nacional que el trabajo en equipo produce mejores resultados?",
        options: [
          "Fomentando competencia individual entre empleados de diferentes áreas",
          "Limitando la colaboración para evitar conflictos de intereses",
          "Evitando la toma de decisiones conjuntas para mantener la eficiencia",
          "Combinando talentos multidisciplinarios y fomentando compromiso con valores"
        ],
        correct: 3,
        explanation: {
          correct: "Cervecería Nacional demuestra que el trabajo en equipo produce mejores resultados al combinar talentos multidisciplinarios (personas de distintas áreas y conocimientos) y fomentar el compromiso con los valores de la empresa.",
          incorrect: [
            "La empresa no fomenta la competencia individual.",
            "La empresa no limita la colaboración.",
            "La empresa no evita la toma de decisiones conjuntas."
          ]
        },
        category: 'equipos'
      },
      {
        id: 11,
        question: "¿Qué caracteriza la comunicación multicanal de Cervecería Nacional?",
        options: [
          "Creación de confusión deliberada entre diferentes canales de comunicación",
          "Limitación del alcance de la información para controlar los mensajes",
          "Aumento de costos operativos sin beneficio claro para la organización",
          "Garantía de transparencia y alineación manteniendo a todos informados"
        ],
        correct: 3,
        explanation: {
          correct: "La comunicación multicanal de Cervecería Nacional garantiza transparencia y mantiene a todos alineados, lo que fortalece la confianza interna y externa, facilita la coordinación entre departamentos y mejora la relación con clientes y sociedad.",
          incorrect: [
            "La comunicación multicanal no crea confusión.",
            "La comunicación multicanal amplía el alcance de la información.",
            "La comunicación multicanal aporta valor a la empresa."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 12,
        question: "¿Cómo supera Cervecería Nacional la barrera de 'información incompleta' en su comunicación?",
        options: [
          "Ignorando los rumores y dejando que se disipen naturalmente",
          "Aumentando la complejidad de los mensajes para evitar malinterpretaciones",
          "Implementando manuales, capacitaciones y comunicación clara",
          "Limitando el acceso a la información para evitar sobrecarga"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional enfrenta la barrera de información incompleta (que provoca rumores) con soluciones como manuales, capacitaciones y comunicación clara.",
          incorrect: [
            "La empresa no ignora los rumores.",
            "Aumentar la complejidad no resuelve el problema.",
            "Limitar el acceso no es la solución adecuada."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 13,
        question: "¿Qué demuestra que Cervecería Nacional aplica los conceptos de Koontz en la práctica real?",
        options: [
          "Aplicación exclusiva de teoría sin conexión con la realidad operativa",
          "Focalización única en resultados financieros sin considerar otros factores",
          "Conexión de conceptos teóricos con la práctica real de la empresa panameña",
          "Ignorancia total de los principios teóricos en favor de la improvisación"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional demuestra que no se queda solo en la teoría del libro, sino que conecta los conceptos de liderazgo, equipos y comunicación con la práctica real de la empresa, mostrando cómo una empresa panameña concreta logra resultados gracias a la forma en que lidera, organiza equipos y maneja su comunicación.",
          incorrect: [
            "La empresa aplica tanto teoría como práctica.",
            "La empresa va más allá de resultados financieros.",
            "La empresa no ignora los principios teóricos."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 14,
        question: "¿Cómo refleja Cervecería Nacional los principios del liderazgo ético en su operación diaria?",
        options: [
          "Priorizando beneficios personales de la gerencia sobre los del equipo",
          "Evitando la responsabilidad por decisiones tomadas colectivamente",
          "Tomando decisiones sin consultar a otros para mantener la eficiencia",
          "Modelando comportamientos y valores deseados en la práctica diaria"
        ],
        correct: 3,
        explanation: {
          correct: "Cervecería Nacional demuestra liderazgo ético al modelar comportamientos y valores deseados en la práctica, como se ve en su compromiso con la sostenibilidad, el bienestar de los colaboradores y el compromiso con la sociedad.",
          incorrect: [
            "La empresa no prioriza beneficios personales.",
            "La empresa no evita la responsabilidad.",
            "La empresa consulta antes de tomar decisiones importantes."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 15,
        question: "¿Qué papel juegan los talleres y espacios de diálogo en el liderazgo participativo de Cervecería Nacional?",
        options: [
          "Servir exclusivamente como eventos sociales sin impacto en decisiones",
          "Crear conflictos innecesarios al permitir opiniones divergentes",
          "Involucrar al equipo en decisiones importantes construyendo soluciones conjuntas",
          "Limitar la participación a solo niveles jerárquicos superiores"
        ],
        correct: 2,
        explanation: {
          correct: "En Cervecería Nacional se hacen talleres y espacios de diálogo con empleados y comunidades para construir soluciones conjuntas, lo que es parte del liderazgo participativo.",
          incorrect: [
            "Los talleres tienen impacto real en las decisiones.",
            "Los conflictos se manejan constructivamente, no se crean innecesariamente.",
            "La participación incluye a todos los niveles, no solo superiores."
          ]
        },
        category: 'liderazgo'
      }
    ],
    dificil: [
      {
        id: 1,
        question: "¿Cómo demuestra Cervecería Nacional que 'lo que Koontz enseña no es teoría abstracta'?",
        options: [
          "Ignorando completamente los principios teóricos en su operación diaria",
          "Aplicando exclusivamente teoría sin adaptarla a la realidad panameña",
          "Conectando conceptos de liderazgo con la práctica real de la empresa",
          "Enseñando teoría a empleados sin implementarla en la práctica"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional aplica la idea de que 'lo que Koontz enseña no es teoría abstracta' al conectar los conceptos de liderazgo, equipos y comunicación explicados en los capítulos 15 y 16 de Koontz, Weihrich y Cannice con la práctica real de la empresa, demostrando que estos conceptos tienen impacto directo en el éxito y prestigio de la organización.",
          incorrect: [
            "La empresa no ignora los principios teóricos.",
            "La empresa adapta la teoría a la realidad práctica.",
            "La empresa implementa la teoría en la práctica."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 2,
        question: "¿Qué evidencia concreta demuestra que el liderazgo con propósito transforma organizaciones según la experiencia de CNP?",
        options: [
          "Reducción drástica de costos operativos sin considerar impacto humano",
          "Aumento exclusivo de producción sin mejorar condiciones laborales",
          "Inspiración a personas para dar lo mejor de sí con visión y valores",
          "Focalización única en competencia sin considerar responsabilidad social"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional refleja liderazgo con propósito al inspirar a las personas a dar lo mejor de sí con visión, valores y bienestar, como se ve en su compromiso con la sostenibilidad, el bienestar de los colaboradores y el programa 'Transforma', lo que hace que la gente se identifique con la empresa.",
          incorrect: [
            "La empresa no reduce costos sin considerar impacto humano.",
            "La empresa mejora condiciones laborales junto con la producción.",
            "La empresa considera responsabilidad social como parte de su propósito."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 3,
        question: "¿Cómo demuestra Cervecería Nacional que la comunicación clara y responsable genera confianza?",
        options: [
          "Aumentando la burocracia para controlar mejor los mensajes internos",
          "Limitando la información interna para evitar filtraciones no autorizadas",
          "Evitando la comunicación con la sociedad para mantener privacidad",
          "Usando comunicación abierta, clara y multicanal para garantizar transparencia"
        ],
        correct: 3,
        explanation: {
          correct: "Cervecería Nacional utiliza una comunicación abierta, clara y multicanal para garantizar transparencia y mantener a todos alineados, lo que demuestra que la comunicación clara y responsable genera confianza tanto dentro de la empresa como en la sociedad.",
          incorrect: [
            "La empresa no aumenta la burocracia.",
            "La empresa no limita la información interna.",
            "La empresa mantiene comunicación con la sociedad."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 4,
        question: "¿Qué estrategias activas utiliza Cervecería Nacional para superar el 'ruido externo' en su comunicación?",
        options: [
          "Ignorando completamente las críticas sociales y presión del entorno",
          "Aumentando la publicidad para contrarrestar cualquier crítica negativa",
          "Implementando comunicación responsable y transparente con la sociedad",
          "Limitando la comunicación externa para evitar más críticas"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional enfrenta el ruido externo (críticas sociales o presión del entorno) con estrategias activas como comunicación responsable y transparente, lo que fortalece la confianza interna y externa.",
          incorrect: [
            "La empresa no ignora las críticas sociales.",
            "La empresa no se limita a aumentar publicidad.",
            "La empresa no limita la comunicación externa."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 5,
        question: "¿Cómo contribuye el enfoque multidisciplinario de los equipos de CNP a la toma de decisiones?",
        options: [
          "Complicando innecesariamente los procesos con demasiadas opiniones",
          "Creando conflictos constantes por diferencias de perspectiva",
          "Enriqueciendo la toma de decisiones y mejorando la creatividad",
          "Ralentizando las decisiones al requerir consenso de todos los miembros"
        ],
        correct: 2,
        explanation: {
          correct: "En Cervecería Nacional, los equipos de trabajo son multidisciplinarios: combinan gente de distintas áreas y conocimientos, lo que enriquece la toma de decisiones, mejora la creatividad y fortalece el compromiso con los valores de la empresa.",
          incorrect: [
            "La diversidad de perspectivas enriquece, no complica.",
            "Los conflictos se manejan constructivamente, no se crean constantemente.",
            "Las decisiones se mejoran, no se ralentizan innecesariamente."
          ]
        },
        category: 'equipos'
      },
      {
        id: 6,
        question: "¿Qué demuestra el ranking de Merco Empresas 2024 sobre la gestión de Cervecería Nacional?",
        options: [
          "Solo buenos resultados financieros sin considerar otros aspectos",
          "Solo buena publicidad sin sustento en la gestión real de la empresa",
          "Solo buenos productos sin relación con la gestión organizacional",
          "Impacto directo del liderazgo, equipos y comunicación en éxito y prestigio"
        ],
        correct: 3,
        explanation: {
          correct: "El hecho de que Cervecería Nacional esté entre las top 5 de Merco Empresas 2024 demuestra que aplicar bien los conceptos de liderazgo, equipos y comunicación no solo es teoría, sino que impacta directamente en el éxito y prestigio de la empresa.",
          incorrect: [
            "No solo refleja resultados financieros.",
            "No solo refleja buena publicidad.",
            "No solo refleja buenos productos."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 7,
        question: "¿Cómo se manifiesta el compromiso con la sociedad en el liderazgo de Cervecería Nacional?",
        options: [
          "Evitando responsabilidades sociales para enfocarse exclusivamente en producción",
          "Priorizando beneficios económicos sobre cualquier consideración social",
          "Integrando responsabilidad social como parte fundamental de su visión",
          "Delegando responsabilidades sociales a organizaciones externas"
        ],
        correct: 2,
        explanation: {
          correct: "En el caso de Cervecería Nacional, el liderazgo no se basa solo en resultados financieros, sino en principios como la sostenibilidad, el bienestar de los colaboradores y el compromiso con la sociedad. Eso hace que la gente se identifique con la empresa y quiera dar lo mejor de sí.",
          incorrect: [
            "La empresa no evita responsabilidades sociales.",
            "La empresa no prioriza solo beneficios económicos.",
            "La empresa asume responsabilidades sociales directamente."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 8,
        question: "¿Qué caracteriza la participación de empleados en la toma de decisiones en Cervecería Nacional?",
        options: [
          "Participación simbólica sin impacto real en las decisiones finales",
          "Exclusión de empleados en decisiones estratégicas importantes",
          "Involucramiento activo en decisiones importantes a través de talleres",
          "Participación limitada a solo empleados de niveles gerenciales"
        ],
        correct: 2,
        explanation: {
          correct: "En Cervecería Nacional se involucra al equipo en las decisiones importantes. Se hacen talleres y espacios de diálogo con empleados y comunidades para construir soluciones conjuntas, lo que es parte del liderazgo participativo.",
          incorrect: [
            "La participación tiene impacto real en las decisiones.",
            "Los empleados están excluidos de decisiones estratégicas.",
            "La participación incluye a empleados de todos los niveles."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 9,
        question: "¿Cómo demuestra Cervecería Nacional que el trabajo en equipo fomenta el compromiso?",
        options: [
          "Imponiendo metas individuales sin considerar el trabajo colectivo",
          "Evitando la colaboración para mantener la competitividad interna",
          "Fortaleciendo el compromiso con los valores de la empresa a través del trabajo en equipo",
          "Limitando la participación en equipos a solo ciertos departamentos"
        ],
        correct: 2,
        explanation: {
          correct: "Cervecería Nacional demuestra que el trabajo en equipo produce mejores resultados al combinar talentos multidisciplinarios y fomentar el compromiso con los valores de la empresa, lo que enriquece la toma de decisiones, mejora la creatividad y fortalece el compromiso.",
          incorrect: [
            "La empresa combina metas individuales y colectivas.",
            "La empresa fomenta la colaboración, no la evita.",
            "La participación en equipos es transversal a todos los departamentos."
          ]
        },
        category: 'equipos'
      },
      {
        id: 10,
        question: "¿Qué papel juega la innovación en el liderazgo transformacional de Cervecería Nacional?",
        options: [
          "Evitando la innovación para mantener procesos tradicionales probados",
          "Limitando la innovación a solo áreas de investigación y desarrollo",
          "Fomentando la innovación como parte integral del liderazgo transformacional",
          "Delegando la innovación exclusivamente a consultores externos"
        ],
        correct: 2,
        explanation: {
          correct: "El presidente Rodrigo Monteiro practica un liderazgo transformacional promoviendo el bienestar de los colaboradores, fomentando la innovación y refuerza la responsabilidad social, como se ve en el programa 'Transforma'.",
          incorrect: [
            "La empresa no evita la innovación.",
            "La innovación no se limita a solo un área.",
            "La innovación es interna y fomentada por el liderazgo."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 11,
        question: "¿Cómo se integra la sostenibilidad en la comunicación organizacional de Cervecería Nacional?",
        options: [
          "Como un tema secundario sin relación con la estrategia de comunicación",
          "Exclusivamente en comunicaciones externas sin impacto interno",
          "Como un pilar fundamental que refuerza la imagen de marca y valores",
          "Solo en campañas publicitarias sin conexión con la operación real"
        ],
        correct: 2,
        explanation: {
          correct: "El Comité de Sostenibilidad garantiza que la empresa actúe de forma responsable con el medio ambiente, y esto se integra en la comunicación organizacional como parte de los valores fundamentales de la empresa.",
          incorrect: [
            "La sostenibilidad es un pilar fundamental, no secundario.",
            "La sostenibilidad impacta tanto interna como externamente.",
            "La sostenibilidad está conectada con la operación real, no solo con publicidad."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 12,
        question: "¿Qué demuestra que Cervecería Nacional aplica efectivamente la toma de decisiones en grupo?",
        options: [
          "Decisiones unilaterales del presidente sin consulta a otros niveles",
          "Participación de empleados, clientes y comunidades en campañas y proyectos",
          "Exclusión de opiniones externas para mantener el control interno",
          "Decisiones basadas únicamente en datos financieros sin considerar otras perspectivas"
        ],
        correct: 1,
        explanation: {
          correct: "En Cervecería Nacional, las técnicas de toma de decisiones en grupo se usan para crear campañas publicitarias o proyectos sociales, incluyendo la participación de empleados, clientes y comunidades.",
          incorrect: [
            "La empresa no toma decisiones unilaterales.",
            "La empresa incluye opiniones externas en su proceso.",
            "La empresa considera múltiples perspectivas, no solo financieras."
          ]
        },
        category: 'comunicacion'
      },
      {
        id: 13,
        question: "¿Cómo refleja el programa 'Transforma' los principios del liderazgo transformacional?",
        options: [
          "Como una iniciativa aislada sin conexión con la visión organizacional",
          "Exclusivamente enfocado en reducir costos de recursos humanos",
          "Como estrategia de competencia con otras empresas del sector",
          "Mejorando calidad de vida de empleados y creando ambiente laboral positivo"
        ],
        correct: 3,
        explanation: {
          correct: "El programa 'Transforma' está diseñado para mejorar la calidad de vida de los empleados y crear un ambiente laboral positivo, lo que es parte del liderazgo transformacional de la empresa que busca motivar a los colaboradores a ir más allá de sus intereses individuales.",
          incorrect: [
            "El programa está conectado con la visión organizacional.",
            "El programa no tiene como objetivo reducir costos.",
            "El programa no es una estrategia de competencia."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 14,
        question: "¿Qué caracteriza la adaptación del estilo de liderazgo situacional en Cervecería Nacional?",
        options: [
          "Mantenimiento riguroso del mismo estilo sin importar las circunstancias",
          "Cambio aleatorio de estilos sin base en las necesidades reales",
          "Adaptación consciente del estilo según la situación y madurez del equipo",
          "Delegación total de la adaptación a consultores externos"
        ],
        correct: 2,
        explanation: {
          correct: "El liderazgo situacional en Cervecería Nacional se adapta a las necesidades cambiantes de los seguidores y las circunstancias, como se vio durante la pandemia cuando la empresa ajustó su forma de trabajar y de tomar decisiones.",
          incorrect: [
            "El liderazgo situacional implica adaptación, no rigidez.",
            "Los cambios no son aleatorios, sino basados en necesidades reales.",
            "La adaptación es interna y liderada por la organización."
          ]
        },
        category: 'liderazgo'
      },
      {
        id: 15,
        question: "¿Cómo demuestra Cervecería Nacional que la comunicación interna fortalece la coordinación?",
        options: [
          "Limitando la comunicación interna para evitar sobrecarga de información",
          "Usando exclusivamente comunicación escrita sin interacción directa",
          "Evitando la comunicación entre departamentos para mantener la especialización",
          "Utilizando aplicaciones internas, correos y reuniones para coordinar el trabajo"
        ],
        correct: 3,
        explanation: {
          correct: "Cervecería Nacional utiliza aplicaciones internas, correos electrónicos y reuniones para coordinar el trabajo y mantener informados a los colaboradores, lo que fortalece la coordinación entre áreas y departamentos.",
          incorrect: [
            "La empresa no limita la comunicación interna.",
            "La empresa utiliza múltiples canales, no solo comunicación escrita.",
            "La empresa fomenta la comunicación entre departamentos."
          ]
        },
        category: 'comunicacion'
      }
    ]
  };

  // Mezclar preguntas y opciones al iniciar o cambiar dificultad/cantidad
  useEffect(() => {
    if (!questions[difficulty]) return;
    // Seleccionar preguntas
    let selectedQs = [...questions[difficulty]].slice(0, totalQuestions);
    // Mezclar el orden de las preguntas seleccionadas
    selectedQs = shuffleArray(selectedQs);
    // Mezclar opciones de cada pregunta
    const finalQs = selectedQs.map(q => {
      const shuffled = shuffleOptions(q.options, q.correct);
      return {
        ...q,
        options: shuffled.options,
        correct: shuffled.correct
      };
    });
    setShuffledQuestions(finalQs);
    setCurrentQuestion(0); // Reinicia el índice de pregunta actual
  }, [difficulty, totalQuestions]); // Mezcla cuando cambia dificultad/cantidad - REMOVIDO gameState

  // ... (resto del código JSX y funciones, sin cambios)
  const handleStart = () => {
    setGameState('playing');
    setCurrentQuestion(0);
    setScore(0);
    setStreak(0);
    setLives(3);
    setSelectedAnswer(null);
    setIsCorrect(null);
    setShowExplanation(false);
    setTimeLeft(30);
    setBadges({
      liderTransformacional: false,
      comunicadorClaro: false,
      equipoMaestro: false,
      maestroCNP: false,
      competente: false
    });
    setAnswers([]);
  };

  const handleAnswerSelect = (answerIndex) => {
    if (gameState !== 'playing') return;
    setSelectedAnswer(answerIndex);
    const currentQuestions = shuffledQuestions;
    const correct = answerIndex === currentQuestions[currentQuestion].correct;
    setIsCorrect(correct);
    setGameState('feedback');
    setShowExplanation(false);
    // Calcula puntos
    let points = 0;
    let newStreak = streak;
    if (correct) {
      points += 100;
      newStreak = streak + 1;
      setStreak(newStreak);
      // Bonus tiempo
      if (timeLeft > 20) {
        points += 25;
      }
      // Bonus racha
      if (newStreak % 3 === 0) {
        points += 50;
      }
    } else {
      points -= 10;
      newStreak = 0;
      setStreak(0);
      setLives(prev => Math.max(0, prev - 1));
    }
    setScore(prev => Math.max(0, prev + points));
    // Guarda la respuesta del usuario
    const newAnswers = [
      ...answers,
      {
        questionId: currentQuestions[currentQuestion].id,
        selected: answerIndex,
        correct: correct,
        category: currentQuestions[currentQuestion].category
      }
    ];
    setAnswers(newAnswers);
    checkBadges(newAnswers, totalQuestions);
  };

  const checkBadges = (userAnswers, totalQuestions) => {
    // Ids por categoría
    const currentQuestions = shuffledQuestions;
    const catLiderazgo = currentQuestions.slice(0, totalQuestions).filter(q => q.category === 'liderazgo').map(q => q.id);
    const catComunicacion = currentQuestions.slice(0, totalQuestions).filter(q => q.category === 'comunicacion').map(q => q.id);
    const catEquipos = currentQuestions.slice(0, totalQuestions).filter(q => q.category === 'equipos').map(q => q.id);
    // Liderazgo
    const liderazgoCorrect = catLiderazgo.length > 0 &&
      catLiderazgo.every(id =>
        userAnswers.find(a => a.questionId === id && a.correct)
      );
    if (liderazgoCorrect) setBadges(prev => ({ ...prev, liderTransformacional: true }));
    // Comunicación
    const comunicacionCorrect = catComunicacion.length > 0 &&
      catComunicacion.every(id =>
        userAnswers.find(a => a.questionId === id && a.correct)
      );
    if (comunicacionCorrect) setBadges(prev => ({ ...prev, comunicadorClaro: true }));
    // Equipos
    const equiposCorrect = catEquipos.length > 0 &&
      catEquipos.every(id =>
        userAnswers.find(a => a.questionId === id && a.correct)
      );
    if (equiposCorrect) setBadges(prev => ({ ...prev, equipoMaestro: true }));
    // Maestría global
    const totalCorrect = userAnswers.filter(a => a.correct).length;
    const percentage = (totalCorrect / userAnswers.length) * 100;
    if (percentage >= 85) {
      setBadges(prev => ({ ...prev, maestroCNP: true }));
    } else if (percentage >= 70) {
      setBadges(prev => ({ ...prev, competente: true }));
    }
  };

  const handleNext = () => {
    const currentQuestions = shuffledQuestions;
    if (currentQuestion + 1 >= totalQuestions || lives <= 0) {
      setGameState('end');
    } else {
      setCurrentQuestion(prev => prev + 1);
      setSelectedAnswer(null);
      setIsCorrect(null);
      setShowExplanation(false);
      setGameState('playing');
      setTimeLeft(30);
    }
  };

  const handleRetry = () => {
    setGameState('difficulty');
    setBadges({
      liderTransformacional: false,
      comunicadorClaro: false,
      equipoMaestro: false,
      maestroCNP: false,
      competente: false
    });
    setAnswers([]);
  };

  // Timer effect
  useEffect(() => {
    if (gameState === 'playing' && timeLeft > 0) {
      const timer = setTimeout(() => setTimeLeft(prev => prev - 1), 1000);
      return () => clearTimeout(timer);
    } else if (gameState === 'playing' && timeLeft === 0) {
      handleAnswerSelect(-1); // Auto-select wrong answer
    }
    // eslint-disable-next-line
  }, [gameState, timeLeft]);

  // Keyboard controls
  useEffect(() => {
    const handleKeyPress = (e) => {
      if (gameState === 'playing') {
        if (e.key === 'a' || e.key === 'A') handleAnswerSelect(0);
        if (e.key === 'b' || e.key === 'B') handleAnswerSelect(1);
        if (e.key === 'c' || e.key === 'C') handleAnswerSelect(2);
        if (e.key === 'd' || e.key === 'D') handleAnswerSelect(3);
      } else if (gameState === 'feedback' && e.key === 'Enter') {
        setShowExplanation(true);
      } else if ((gameState === 'feedback' || gameState === 'end') && e.key === 'Enter') {
        handleNext();
      }
    };
    window.addEventListener('keydown', handleKeyPress);
    return () => window.removeEventListener('keydown', handleKeyPress);
    // eslint-disable-next-line
  }, [gameState, currentQuestion, selectedAnswer]);

  const getBadgeIcon = (badgeType) => {
    switch (badgeType) {
      case 'liderTransformacional':
        return <Heart className="w-6 h-6" />;
      case 'comunicadorClaro':
        return <MessageSquare className="w-6 h-6" />;
      case 'equipoMaestro':
        return <Users className="w-6 h-6" />;
      case 'maestroCNP':
        return <Trophy className="w-6 h-6" />;
      case 'competente':
        return <Zap className="w-6 h-6" />;
      default:
        return <Trophy className="w-6 h-6" />;
    }
  };

  const getBadgeColor = (badgeType) => {
    switch (badgeType) {
      case 'maestroCNP':
        return 'bg-yellow-400 text-gray-900';
      case 'competente':
        return 'bg-blue-400 text-white';
      default:
        return 'bg-green-400 text-white';
    }
  };

  // UI
  if (gameState === 'welcome') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full text-center">
          <div className="mb-6">
            <h1 className="text-3xl font-bold text-gray-800 mb-4">Líderes & Equipos: El Reto CNP</h1>
            <p className="text-gray-600 mb-6">Refuerza tus conocimientos sobre liderazgo, equipos y comunicación en Cervecería Nacional de Panamá</p>
            <div className="bg-gray-100 rounded-lg p-4 text-left mb-6">
              <h2 className="font-semibold text-gray-800 mb-2">Objetivo del Reto</h2>
              <p className="text-sm text-gray-700">
                Aplicar los conceptos de liderazgo, equipos y comunicación explicados en los capítulos 15 y 16 de Koontz, Weihrich y Cannice, conectando esos conceptos con la práctica real de Cervecería Nacional de Panamá.
              </p>
            </div>
            <button
              onClick={() => setGameState('playerInfo')}
              className="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200"
            >
              Comenzar
            </button>
          </div>
        </div>
      </div>
    );
  }

  if (gameState === 'playerInfo') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full text-center">
          <div className="mb-6">
            <h1 className="text-3xl font-bold text-gray-800 mb-2">¡Bienvenido!</h1>
            <p className="text-gray-600 mb-6">Ingresa tu nombre para comenzar el reto</p>
            <input
              type="text"
              value={playerName}
              onChange={(e) => setPlayerName(e.target.value)}
              placeholder="Tu nombre"
              className="w-full p-3 border border-gray-300 rounded-lg mb-6 text-center text-lg"
              onKeyPress={(e) => e.key === 'Enter' && playerName.trim() && setGameState('difficulty')}
            />
            <button
              onClick={() => playerName.trim() && setGameState('difficulty')}
              disabled={!playerName.trim()}
              className={`w-full font-semibold py-3 px-6 rounded-lg transition-colors duration-200 ${
                playerName.trim()
                  ? 'bg-emerald-600 hover:bg-emerald-700 text-white'
                  : 'bg-gray-300 text-gray-500 cursor-not-allowed'
              }`}
            >
              Continuar
            </button>
          </div>
        </div>
      </div>
    );
  }

  if (gameState === 'difficulty') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full text-center">
          <div className="mb-6">
            <h1 className="text-2xl font-bold text-gray-800 mb-2">¡Hola {playerName}!</h1>
            <p className="text-gray-600 mb-6">Selecciona el nivel de dificultad y número de preguntas</p>
            <div className="mb-6">
              <h3 className="font-semibold text-gray-800 mb-3">Nivel de Dificultad</h3>
              <div className="space-y-3">
                {[
                  { value: 'basico', label: 'Básico', desc: 'Preguntas fundamentales sobre liderazgo y equipos' },
                  { value: 'medio', label: 'Medio', desc: 'Preguntas intermedias con aplicaciones prácticas' },
                  { value: 'dificil', label: 'Difícil', desc: 'Preguntas avanzadas y casos reales' }
                ].map((level) => (
                  <button
                    key={level.value}
                    onClick={() => setDifficulty(level.value)}
                    className={`w-full text-left p-4 rounded-lg transition-all duration-200 ${
                      difficulty === level.value
                        ? 'bg-emerald-100 border-2 border-emerald-500'
                        : 'bg-gray-50 border border-gray-200 hover:bg-emerald-50'
                    }`}
                  >
                    <div className="font-semibold text-gray-800">{level.label}</div>
                    <div className="text-sm text-gray-600">{level.desc}</div>
                  </button>
                ))}
              </div>
            </div>
            <div className="mb-6">
              <h3 className="font-semibold text-gray-800 mb-3">Número de Preguntas</h3>
              <div className="flex justify-center space-x-4">
                {[12, 15].map((num) => (
                  <button
                    key={num}
                    onClick={() => setTotalQuestions(num)}
                    className={`w-16 h-16 rounded-full flex items-center justify-center text-xl font-bold transition-all duration-200 ${
                      totalQuestions === num
                        ? 'bg-emerald-600 text-white'
                        : 'bg-gray-200 text-gray-700 hover:bg-emerald-200'
                    }`}
                  >
                    {num}
                  </button>
                ))}
              </div>
              <p className="text-sm text-gray-600 mt-2">Selecciona 12 o 15 preguntas</p>
            </div>
            <button
              onClick={handleStart}
              className="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200"
            >
              ¡Empezar Reto!
            </button>
          </div>
        </div>
      </div>
    );
  }

  if (gameState === 'playing') {
    const currentQuestions = shuffledQuestions;
    const question = currentQuestions[currentQuestion];
    // Agregar una verificación para evitar errores si shuffledQuestions aún no está listo
    if (!question) {
      return <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4"><div className="bg-white rounded-xl shadow-lg p-8 text-center">Cargando preguntas...</div></div>;
    }
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 p-4">
        <div className="max-w-2xl mx-auto">
          {/* Header */}
          <div className="bg-white rounded-xl shadow-lg p-4 mb-6">
            <div className="flex justify-between items-center mb-2">
              <div className="text-sm font-medium text-gray-600">
                {playerName} • Pregunta {currentQuestion + 1} de {totalQuestions}
              </div>
              <div className="flex items-center space-x-4">
                <div className="flex items-center text-sm">
                  <Clock className="w-4 h-4 mr-1 text-gray-600" />
                  <span className="font-mono">{timeLeft}s</span>
                </div>
                <div className="flex items-center text-sm">
                  <Heart className="w-4 h-4 mr-1 text-red-500" />
                  <span>{lives}</span>
                </div>
                <div className="text-sm font-bold text-emerald-600">
                  {score} pts
                </div>
              </div>
            </div>
            <div className="w-full bg-gray-200 rounded-full h-2">
              <div
                className="bg-emerald-600 h-2 rounded-full transition-all duration-300"
                style={{ width: `${((currentQuestion + 1) / totalQuestions) * 100}%` }}
              ></div>
            </div>
          </div>
          {/* Question */}
          <div className="bg-white rounded-xl shadow-lg p-6 mb-6">
            <h2 className="text-xl font-semibold text-gray-800 mb-6 leading-relaxed">
              {question.question}
            </h2>
            <div className="space-y-3">
              {question.options.map((option, index) => (
                <button
                  key={index}
                  onClick={() => handleAnswerSelect(index)}
                  className="w-full text-left p-4 bg-gray-50 hover:bg-emerald-50 border-2 border-gray-200 hover:border-emerald-300 rounded-lg transition-all duration-200 text-gray-700 font-medium"
                >
                  <div className="flex items-center">
                    <span className="w-8 h-8 bg-emerald-100 text-emerald-700 rounded-full flex items-center justify-center font-bold mr-3">
                      {String.fromCharCode(65 + index)}
                    </span>
                    <span>{option}</span>
                  </div>
                </button>
              ))}
            </div>
          </div>
          <div className="text-center text-sm text-gray-500">
            <p>Presiona A, B, C o D para responder • {streak > 0 && `Racha: ${streak}`}</p>
          </div>
        </div>
      </div>
    );
  }

  if (gameState === 'feedback') {
    const currentQuestions = shuffledQuestions;
    const question = currentQuestions[currentQuestion];
    // Agregar una verificación para evitar errores si shuffledQuestions aún no está listo
    if (!question) {
      return <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4"><div className="bg-white rounded-xl shadow-lg p-8 text-center">Cargando feedback...</div></div>;
    }
    const pointsEarned = isCorrect ?
      (100 + (timeLeft > 20 ? 25 : 0) + (streak % 3 === 0 && streak > 0 ? 50 : 0)) : -10;
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 p-4">
        <div className="max-w-2xl mx-auto">
          {/* Header */}
          <div className="bg-white rounded-xl shadow-lg p-4 mb-6">
            <div className="flex justify-between items-center">
              <div className="text-sm font-medium text-gray-600">
                {playerName} • Pregunta {currentQuestion + 1} de {totalQuestions}
              </div>
              <div className="text-sm font-bold text-emerald-600">
                {score} pts
              </div>
            </div>
          </div>
          {/* Feedback Animation */}
          <div className="bg-white rounded-xl shadow-lg p-8 mb-6 text-center relative overflow-hidden">
            {isCorrect ? (
              <div className="animate-bounce">
                <Check className="w-16 h-16 text-green-500 mx-auto mb-4" />
                <h3 className="text-2xl font-bold text-green-600 mb-2">¡Correcto!</h3>
                {streak % 3 === 0 && streak > 0 && (
                  <div className="bg-yellow-100 border border-yellow-300 rounded-lg p-2 mb-4 inline-block">
                    <Trophy className="w-6 h-6 text-yellow-600 inline mr-2" />
                    <span className="text-yellow-700 font-semibold">¡Racha de {streak}! +50 pts</span>
                  </div>
                )}
              </div>
            ) : (
              <div className="animate-pulse">
                <X className="w-16 h-16 text-red-500 mx-auto mb-4" />
                <h3 className="text-2xl font-bold text-red-600 mb-2">Incorrecto</h3>
                <p className="text-gray-600 mb-2">La respuesta correcta era:</p>
                <p className="font-semibold text-emerald-600">
                  {question.options[question.correct]}
                </p>
              </div>
            )}
            {/* Points */}
            <div className={`text-lg font-bold mb-4 ${isCorrect ? 'text-green-600' : 'text-red-600'}`}>
              {isCorrect ? '+' : ''}{pointsEarned} puntos
            </div>
            {/* Explanation */}
            {showExplanation ? (
              <div className="text-left bg-gray-50 rounded-lg p-4 mt-4">
                <h4 className="font-semibold text-gray-800 mb-2">Explicación:</h4>
                <p className="text-gray-700 mb-3">{question.explanation.correct}</p>
                {!isCorrect && (
                  <div className="mt-3">
                    <h5 className="font-semibold text-gray-800 mb-2">Por qué las otras opciones son incorrectas:</h5>
                    {question.explanation.incorrect.map((reason, index) => (
                      <p key={index} className="text-gray-600 text-sm mb-1">
                        <span className="font-medium">{String.fromCharCode(65 + index)}:</span> {reason}
                      </p>
                    ))}
                  </div>
                )}
              </div>
            ) : (
              <button
                onClick={() => setShowExplanation(true)}
                className="mt-4 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-2 px-6 rounded-lg transition-colors duration-200"
              >
                Ver explicación
              </button>
            )}
          </div>
          {showExplanation && (
            <button
              onClick={handleNext}
              className="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200"
            >
              Siguiente
            </button>
          )}
        </div>
      </div>
    );
  }

  if (gameState === 'end') {
    const totalCorrect = answers.filter(a => a.correct).length;
    const percentage = Math.round((totalCorrect / totalQuestions) * 100);
    const earnedBadges = Object.entries(badges).filter(([_, earned]) => earned);
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-xl p-8 max-w-md w-full">
          <div className="text-center mb-6">
            <h1 className="text-3xl font-bold text-gray-800 mb-2">¡Felicidades {playerName}!</h1>
            <div className="text-4xl font-bold text-emerald-600 mb-2">{score} puntos</div>
            <div className="text-lg text-gray-600">
              {percentage}% de respuestas correctas
            </div>
          </div>
          {/* Performance Level */}
          <div className="bg-gray-50 rounded-lg p-4 mb-6 text-center">
            {percentage >= 85 ? (
              <div>
                <Trophy className="w-8 h-8 text-yellow-500 mx-auto mb-2" />
                <p className="font-semibold text-yellow-700">Excelente - Maestro CNP</p>
              </div>
            ) : percentage >= 70 ? (
              <div>
                <Zap className="w-8 h-8 text-blue-500 mx-auto mb-2" />
                <p className="font-semibold text-blue-700">Bueno - Competente</p>
              </div>
            ) : (
              <div>
                <p className="font-semibold text-gray-700">Revisión recomendada</p>
              </div>
            )}
          </div>
          {/* Badges */}
          {earnedBadges.length > 0 && (
            <div className="mb-6">
              <h3 className="font-semibold text-gray-800 mb-3 text-center">Insignias ganadas</h3>
              <div className="flex flex-wrap justify-center gap-2">
                {earnedBadges.map(([badgeType]) => (
                  <div
                    key={badgeType}
                    className={`flex items-center px-3 py-2 rounded-lg ${getBadgeColor(badgeType)}`}
                  >
                    {getBadgeIcon(badgeType)}
                    <span className="ml-2 font-medium text-sm capitalize">
                      {badgeType.replace(/([A-Z])/g, ' $1').trim()}
                    </span>
                  </div>
                ))}
              </div>
            </div>
          )}
          {/* Actions */}
          <div className="space-y-3">
            <button
              onClick={handleRetry}
              className="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200"
            >
              Reintentar
            </button>
            <button
              onClick={() => setGameState('difficulty')}
              className="w-full bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-3 px-6 rounded-lg transition-colors duration-200"
            >
              Cambiar dificultad
            </button>
            <button
              onClick={() => setGameState('welcome')}
              className="w-full bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-3 px-6 rounded-lg transition-colors duration-200"
            >
              Menú principal
            </button>
          </div>
        </div>
      </div>
    );
  }

  return null;
};

export default App;