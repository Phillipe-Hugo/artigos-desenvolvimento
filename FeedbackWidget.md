# FeedbackWidget

Componente React para coleta de feedback sobre a utilidade de uma página ou documento. Permite ao usuário registrar se o conteúdo foi útil (positivo) ou não (negativo), exibindo mensagens dinâmicas de agradecimento e tratamento de erros.

---

## Sumário

- [Descrição Geral](#descrição-geral)
- [Props](#props)
- [Funcionamento](#funcionamento)
- [Fluxo de Feedback](#fluxo-de-feedback)
- [Internacionalização](#internacionalização)
- [Persistência no LocalStorage](#persistência-no-localstorage)
- [Dependências](#dependências)
- [Exemplo de Uso](#exemplo-de-uso)
- [Código Fonte](#código-fonte)
- [Customização](#customização)

---

## Descrição Geral

O `FeedbackWidget` é um componente React que exibe botões para o usuário indicar se achou útil uma página ou documento. A resposta é persistida localmente para evitar múltiplos envios e registrada em uma ação backend. As mensagens e labels são internacionalizadas e podem ser customizadas via dicionário.

---

## Props

```typescript
interface FeedbackWidgetProps {
  documentoId: string; // Identificador único do documento/página
  lang: Locale;        // Código do idioma (ex: 'pt', 'en', etc)
}
```

---

## Funcionamento

- Ao renderizar, o componente busca um dicionário de textos via React Query e exibe a pergunta correspondente.
- Permite ao usuário clicar em "Sim/Não" (ou "Yes/No") para enviar o feedback.
- Após o envio, exibe uma mensagem de agradecimento adequada ao tipo de feedback.
- Feedback é persistido no `localStorage` para impedir reenvio do mesmo usuário para o mesmo documento.

---

## Fluxo de Feedback

1. **Renderização Inicial**: Pergunta se a página foi útil. Busca textos do dicionário.
2. **Envio de Feedback**: Ao clicar em "Sim" ou "Não", chama a função `insertOrUpdateFeedbackScore`.
3. **Persistência**: Salva o tipo de feedback no `localStorage` usando a chave `feedback-documento-{documentoId}`.
4. **Status e Mensagens**:
   - Mostra loading enquanto envia.
   - Após envio, exibe agradecimento personalizado.
   - Em caso de erro, mostra mensagem de erro via toast.

---

## Internacionalização

O componente utiliza o dicionário carregado para exibir textos no idioma selecionado (`lang`). Caso o dicionário não esteja disponível, utiliza os textos padrão em português ou inglês.

---

## Persistência no LocalStorage

- Feedback é salvo por documento, impedindo múltiplos envios para o mesmo `documentoId` pelo mesmo usuário.
- Chave utilizada: `feedback-documento-{documentoId}`.
- Estado do feedback é recuperado no carregamento do componente.

---

## Dependências

- **React** (`useState`)
- **React Query** (`useQuery`)
- **Sonner** (`toast`)
- **Lucide React** (`ThumbsUp`, `ThumbsDown`)
- **Componentes customizados** (`Button`)
- **Funções customizadas**:
  - `insertOrUpdateFeedbackScore`
  - `getDictionaryAction`

---

## Exemplo de Uso

```tsx
import FeedbackWidget from '@/components/FeedbackWidget';

<FeedbackWidget documentoId="abc123" lang="pt" />
```

---

## Código Fonte

```typescript
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
```

---

## Customização

- Textos podem ser customizados via dicionário retornado por `getDictionaryAction`.
- Aparência dos botões pode ser adaptada via props do componente `Button` e classes Tailwind.

---

## Estrutura do Dicionário Esperada

```typescript
type FeedbackDictionary = {
  question: string;
  yes: string;
  no: string;
  sending: string;
  success: string;
  error: string;
  thanks_positive: string;
  thanks_negative: string;
};
```

---

## Observações

- O componente só renderiza feedback customizado se o dicionário estiver disponível.
- Pode ser facilmente adaptado para outros contextos alterando os textos e integrando com outros sistemas de registro de feedback.

---

## Autor

Documentação gerada por [Phillipe-Hugo](https://github.com/Phillipe-Hugo).
