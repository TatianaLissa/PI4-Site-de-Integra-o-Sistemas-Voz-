# PI4-Site-de-Integra-o-Sistemas-Voz-
Um Site com integração de API React 
npm install gh-pages --save-dev
"homepage": "https://<TatianaLissa>.github.io/<PI4-Site-de- Integração-Sistemas-Voz>",
    "scripts": {
      "predeploy": "npm run build",
      "deploy": "gh-pages -d build"
    }
    npm run deploy

    import React, { useState, useRef, useCallback } from 'react';

// Hook useTextToSpeech que integra a síntese de fala e controle de fonemas
export function useTextToSpeech(text) {
  const [isSpeaking, setIsSpeaking] = useState(false);
  const [isPaused, setIsPaused] = useState(false);
  const [volume, setVolume] = useState(1);
  const [rate, setRate] = useState(1);
  const [phonemes, setPhonemes] = useState(null); // Estado para armazenar fonemas
  const utteranceRef = useRef(null);

  // Função simulada para chamar um serviço externo para conversão texto-para-fonema
  const fetchPhonemes = useCallback(async (text) => {
    // Simulação: chamada a API REST fictícia
    // Aqui deve substituir para chamada real via fetch ou axios
    const fakeApiResponse = {
      phonemes: "pʰ oʊ n iː m z" // exemplo fictício
    };
    setPhonemes(fakeApiResponse.phonemes);
    return fakeApiResponse.phonemes;
  }, []);

  const speak = useCallback(async () => {
    if (!text) return;

    // Obter fonemas do serviço externo
    await fetchPhonemes(text);

    if (utteranceRef.current) {
      window.speechSynthesis.cancel();
      utteranceRef.current = null;
    }

    const utterance = new SpeechSynthesisUtterance(text);
    utterance.volume = volume;
    utterance.rate = rate;

    utterance.onstart = () => setIsSpeaking(true);
    utterance.onend = () => {
      setIsSpeaking(false);
      setIsPaused(false);
    };
    utterance.onpause = () => setIsPaused(true);
    utterance.onresume = () => setIsPaused(false);
    utterance.onerror = () => {
      setIsSpeaking(false);
      setIsPaused(false);
    };

    utterance.onboundary = event => {
      console.log(`Boundary event: charIndex=${event.charIndex}, name=${event.name}`);
    };

    utteranceRef.current = utterance;
    window.speechSynthesis.speak(utterance);
  }, [text, volume, rate, fetchPhonemes]);

  const pause = useCallback(() => {
    if (window.speechSynthesis.speaking && !window.speechSynthesis.paused) {
      window.speechSynthesis.pause();
      setIsPaused(true);
    }
  }, []);

  const resume = useCallback(() => {
    if (window.speechSynthesis.paused) {
      window.speechSynthesis.resume();
      setIsPaused(false);
    }
  }, []);

  const stop = useCallback(() => {
    window.speechSynthesis.cancel();
    utteranceRef.current = null;
    setIsSpeaking(false);
    setIsPaused(false);
  }, []);

  return {
    isSpeaking,
    isPaused,
    volume,
    setVolume,
    rate,
    setRate,
    phonemes, // retornando fonemas
    speak,
    pause,
    resume,
    stop
  };
}

// Componente TextToSpeechControl que renderiza os controles e mostra fonemas
export function TextToSpeechControl({ text }) {
  const {
    isSpeaking,
    isPaused,
    volume,
    setVolume,
    rate,
    setRate,
    phonemes,
    speak,
    pause,
    resume,
    stop
  } = useTextToSpeech(text);

  const handlePlayPause = () => {
    if (isSpeaking) {
      if (isPaused) {
        resume();
      } else {
        pause();
      }
    } else {
      speak();
    }
  };

  return (
    <div>
      <button onClick={handlePlayPause}>
        {isSpeaking ? (isPaused ? 'Retomar' : 'Pausar') : 'Tocar'}
      </button>
      <button onClick={stop} disabled={!isSpeaking}>
        Parar
      </button>
      <div>
        <label>
          Volume: {Math.round(volume * 100)}%
          <input
            type="range"
            min="0"
            max="1"
            step="0.01"
            value={volume}
            onChange={e => setVolume(Number(e.target.value))}
            disabled={isSpeaking && !isPaused}
          />
        </label>
      </div>
      <div>
        <label>
          Velocidade: {rate.toFixed(2)}
          <input
            type="range"
            min="0.5"
            max="2"
            step="0.1"
            value={rate}
            onChange={e => setRate(Number(e.target.value))}
            disabled={isSpeaking && !isPaused}
          />
        </label>
      </div>
      {phonemes && (
        <div>
          <strong>Fonemas:</strong> {phonemes}
        </div>
      )}
    </div>
  );
}

export default TextToSpeechControl;
