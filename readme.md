# README — Estudo de caso (D2L §9.5) — Respostas das Questões

## 1) Efeito de `num_examples` em `load_data_nmt` sobre os tamanhos dos vocabulários (fonte e alvo)

Foram avaliados diferentes valores do parâmetro `num_examples` em `load_data_nmt`, com registro de **amostras** de tensores (`X` = fonte/EN, `Y` = alvo/FR) e respectivos “valid lengths”.  
Os tensores não fornecem diretamente `|V_src|` e `|V_tgt|`, porém os **IDs de tokens observados** permitem estimar **limites inferiores** para os tamanhos de vocabulário (pois o tamanho deve ser ≥ 1 + maior ID observado).

A seguir apresenta-se uma **tabela comparativa** que utiliza o **maior ID observado** em cada amostra como **proxy** (limite inferior) para `|V_src|` e `|V_tgt|`.

> Observação: os tamanhos reais de vocabulário tendem a ser **maiores** do que esses limites, pois a análise considera apenas uma **amostra** de um *batch*. Ainda assim, a tendência geral é informativa.

| `num_examples` | Maior ID em **X** (fonte) | Limite inferior `|V_src|` | Maior ID em **Y** (alvo) | Limite inferior `|V_tgt|` |
|---|---:|---:|---:|---:|
| 100  | 37  | ≥ 38  | 29   | ≥ 30  |
| 300  | 97  | ≥ 98  | 106  | ≥ 107 |
| 900  | 226 | ≥ 227 | 135  | ≥ 136 |
| 2700 | 243 | ≥ 244 | 616  | ≥ 617 |

### Interpretação dos resultados
- **Tendência geral:** com o aumento de `num_examples`, **novos tipos** (palavras distintas) são observados, elevando os tamanhos de vocabulário. Tal comportamento é esperado em corpora naturais (efeito tipo–token; distribuição de Zipf).
- **Retornos decrescentes:** o crescimento de `|V|` tende a **desacelerar** em corpora maiores (termos frequentes surgem cedo; novos tipos aparecem com menor frequência posteriormente).
- **Variações entre amostras:** diferenças aparentes decorrem do uso de **apenas uma amostra** do *batch* para inferência. No ponto de 2700 exemplos, por exemplo, observou-se ID elevado em `Y` (até 616), sugerindo vocabulário-alvo mais amplo — hipótese que, idealmente, deve ser **confirmada** por contagem global do vocabulário (considerando `min_freq` e *reserved tokens*).
- **Aspecto técnico:** com *tokens* reservados (`<pad>`, `<bos>`, `<eos>`) e `min_freq=2` (configuração comum na seção), o vocabulário efetivo depende de: (i) frequências no recorte (`num_examples`), (ii) limiar de frequência mínima e (iii) inclusão dos *reserved tokens*.

**Conclusão:** o aumento de `num_examples` implica **elevação de `|V_src|` e `|V_tgt|`**, com **redução do ritmo de crescimento** à medida que o corpus se expande. As estimativas via “maior ID observado” evidenciam essa tendência mesmo sem a contagem exata de tipos.

---

## 2) Adequação da tokenização em nível de palavra para línguas sem separadores (chinês/japonês)

**Síntese:** em geral, **não** é recomendável.  
**Justificativa:** em chinês e japonês **não há separadores explícitos de palavras** (p. ex., espaço). A tokenização por palavra baseada em espaços é inaplicável; além disso, segmentadores por dicionário podem introduzir **ambiguidade**, elevar o **OOV** (*out-of-vocabulary*) e depender de **regras/recursos externos**.

**Alternativas recomendadas:**
- **Subpalavras** (BPE, WordPiece, Unigram/SentencePiece):  
  Reduzem OOV, capturam regularidades morfológicas e **independem de espaços**; constituem prática corrente em NMT/LLMs.
- **Nível de caracteres:**  
  No chinês, cada caractere possui carga semântica. Modelos por caracteres podem ser eficazes, ainda que gerem **sequências mais longas** e exijam maior capacidade de modelagem de contexto.

**Consonância com o D2L §9.5:** a seção adota tokenização em palavras por motivos didáticos, mas registra que **modelos de estado da arte empregam tokenizações mais avançadas** (notadamente, subpalavras).

---

### Referências essenciais
- Sennrich, R.; Haddow, B.; Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units (BPE).*  
- Kudo, T. (2018). *SentencePiece: A simple and language independent subword tokenizer and detokenizer for Neural Text Processing.*  
- *Dive into Deep Learning*, §9.5 — discussão sobre vocabulário e menção ao uso de tokenização avançada em modelos SOTA.
