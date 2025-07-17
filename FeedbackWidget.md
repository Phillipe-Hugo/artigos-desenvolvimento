'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { ThumbsUp, ThumbsDown } from 'lucide-react';
import { insertOrUpdateFeedbackScore } from '@/actions/feedback';
import { toast } from 'sonner';
import type { Locale } from '@/i18n.config';
import { useQuery } from '@tanstack/react-query';
import { getDictionaryAction } from '@/actions/dictionary-action';

interface FeedbackWidgetProps {
  documentoId: string;
  lang: Locale;
}

export default function FeedbackWidget({ documentoId, lang }: FeedbackWidgetProps) {
  const { data: dictionary } = useQuery({
    queryKey: ['dictionary', lang],
    queryFn: () => getDictionaryAction(lang)
  });

  const feedback = dictionary?.feedback;

  const chaveLocalStorage = `feedback-documento-${documentoId}`;
  const [feedbackEnviado, setFeedbackEnviado] = useState<boolean>(() => {
    if (typeof window !== 'undefined') {
      return localStorage.getItem(chaveLocalStorage) !== null;
    }
    return false;
  });
  const [carregandoFeedback, setCarregandoFeedback] = useState<boolean>(false);
  const [tipoFeedback, setTipoFeedback] = useState<'positivo' | 'negativo' | null>(() => {
    if (typeof window !== 'undefined') {
      const feedbackSalvo = localStorage.getItem(chaveLocalStorage);
      return feedbackSalvo as 'positivo' | 'negativo' | null;
    }
    return null;
  });

  const enviarFeedback = async (tipo: 'positivo' | 'negativo') => {
    if (feedbackEnviado) return;
    
    setCarregandoFeedback(true);
    
    try {
      const resultado = await insertOrUpdateFeedbackScore(documentoId, tipo === 'positivo');
      
      if (resultado.success) {
        localStorage.setItem(chaveLocalStorage, tipo);
        setFeedbackEnviado(true);
        setTipoFeedback(tipo);
        
        toast.success(feedback?.success || 'Feedback enviado com sucesso!');
      } else {
        toast.error(resultado.message || (feedback?.error || 'Erro ao enviar feedback'));
      }
    } catch (error) {
      console.error('Erro ao enviar feedback:', error);
      toast.error(feedback?.error || 'Erro ao enviar feedback. Tente novamente.');
    } finally {
      setCarregandoFeedback(false);
    }
  };

  if (!feedback) {
    return (
      <div className="border-t pt-8 mt-8">
        <div className="flex flex-col items-center gap-4">
          <div className="flex flex-row items-center justify-center gap-4">
            <p className="text-lg text-gray-800">
              {lang === 'pt' ? 'Esta página foi útil?' : 'Did this page help you?'}
            </p>
            
            {!feedbackEnviado ? (
              <div className="flex gap-3">
                <Button
                  variant="outline"
                  size="sm"
                  onClick={() => enviarFeedback('positivo')}
                  disabled={carregandoFeedback}
                  className="flex items-center gap-2 rounded-full border-blue-500 text-blue-700 hover:bg-blue-50 hover:text-blue-700"
                >
                  <ThumbsUp className="w-4 h-4" />
                  {carregandoFeedback ? (lang === 'pt' ? 'Enviando...' : 'Sending...') : (lang === 'pt' ? 'Sim' : 'Yes')}
                </Button>
                
                <Button
                  variant="outline"
                  size="sm"
                  onClick={() => enviarFeedback('negativo')}
                  disabled={carregandoFeedback}
                  className="flex items-center gap-2 rounded-full border-blue-500 text-blue-700 hover:bg-blue-50 hover:text-blue-700"
                >
                  <ThumbsDown className="w-4 h-4" />
                  {carregandoFeedback ? (lang === 'pt' ? 'Enviando...' : 'Sending...') : (lang === 'pt' ? 'Não' : 'No')}
                </Button>
              </div>
            ) : (
              <div className="flex flex-col items-center gap-2">
                <div className="flex items-center gap-2 text-sm text-gray-600">
                  {tipoFeedback === 'positivo' ? (
                    <>
                      <ThumbsUp className="w-4 h-4 text-green-600" />
                      <span>{lang === 'pt' ? 'Obrigado pelo seu feedback positivo!' : 'Thank you for your positive feedback!'}</span>
                    </>
                  ) : (
                    <>
                      <ThumbsDown className="w-4 h-4 text-red-600" />
                      <span>{lang === 'pt' ? 'Obrigado pelo seu feedback. Vamos melhorar!' : 'Thank you for your feedback. We will improve!'}</span>
                    </>
                  )}
                </div>
              </div>
            )}
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="border-t pt-8 mt-8">
      <div className="flex flex-col items-center gap-4">
        <div className="flex flex-row items-center justify-center gap-4">
          <p className="text-lg text-gray-800">
            {feedback.question}
          </p>
          
          {!feedbackEnviado ? (
            <div className="flex gap-3">
              <Button
                variant="outline"
                size="sm"
                onClick={() => enviarFeedback('positivo')}
                disabled={carregandoFeedback}
                className="flex items-center gap-2 rounded-full border-blue-500 text-blue-700 hover:bg-blue-50 hover:text-blue-700"
              >
                <ThumbsUp className="w-4 h-4" />
                {carregandoFeedback ? feedback.sending : feedback.yes}
              </Button>
              
              <Button
                variant="outline"
                size="sm"
                onClick={() => enviarFeedback('negativo')}
                disabled={carregandoFeedback}
                className="flex items-center gap-2 rounded-full border-blue-500 text-blue-700 hover:bg-blue-50 hover:text-blue-700"
              >
                <ThumbsDown className="w-4 h-4" />
                {carregandoFeedback ? feedback.sending : feedback.no}
              </Button>
            </div>
          ) : (
            <div className="flex flex-col items-center gap-2">
              <div className="flex items-center gap-2 text-sm text-gray-600">
                {tipoFeedback === 'positivo' ? (
                  <>
                    <ThumbsUp className="w-4 h-4 text-green-600" />
                    <span>{feedback.thanks_positive}</span>
                  </>
                ) : (
                  <>
                    <ThumbsDown className="w-4 h-4 text-red-600" />
                    <span>{feedback.thanks_negative}</span>
                  </>
                )}
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}
