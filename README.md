# AP1 - Visao Computacional: Deteccao de Telefones

Projeto academico de Visao Computacional para detectar telefones celulares em webcams e classificar situacoes de risco (THREAT) com base em regras geometricas e logica temporal.

## Objetivo

Construir um pipeline completo de deteccao de objetos com YOLOv8 para:

1. Preparar e converter dataset anotado em Pascal VOC para formato YOLO.
2. Treinar um detector para a classe `mobile_phone`.
3. Avaliar o modelo com metricas de deteccao.
4. Aplicar regra de negocio para classificar deteccoes como seguras ou ameaca.
5. Rodar inferencia em lote e em tempo real (video/webcam).
6. Exportar modelo para ONNX e medir desempenho.

## Escopo do Trabalho

O notebook principal cobre um fluxo de ponta a ponta:

- Instalacao de dependencias.
- Extracao e organizacao de datasets compactados.
- Conversao de anotacoes XML (Pascal VOC) para TXT (YOLO).
- Divisao treino/validacao.
- Treinamento com YOLOv8n.
- Pos-processamento com regra customizada para deteccao de tentativa de foto.
- Avaliacao robusta (mAP, precision, recall, sweep de threshold por F1).
- Inferencia em video/webcam com filtro temporal anti-flicker.
- Coleta de hard negatives.
- Exportacao para ONNX e benchmark de latencia/FPS.

## Estrutura do Repositorio

- `AP1_Visao_Computacional_Detecção_de_Telefones.ipynb`: notebook principal com pipeline completo.
- `Collab_AP1_COMPUTER_VISION_Detecção_de_Telefones_Breno.ipynb`: versao alternativa/colaborativa do experimento.
- `README.md`: documentacao do projeto.

## Dataset

O fluxo foi preparado para consumir arquivos ZIP com imagens e anotacoes.

Entradas esperadas no ambiente do notebook (exemplo em Colab):

- `/content/100_imagens_celulares.zip`
- `/content/520_imagens_celulares.zip`
- `/content/1300_imagens_celulares.zip`

Processamento realizado:

1. Extrai os ZIPs para um diretorio base.
2. Busca imagens (`.jpg`/`.png`) e anotacoes XML recursivamente.
3. Converte bounding boxes para formato YOLO normalizado.
4. Gera split 80/20 em:
	- `images/train`, `images/val`
	- `labels/train`, `labels/val`
5. Cria `data.yaml` com 1 classe:
	- `0: mobile_phone`

## Metodologia

### 1. Treinamento

Modelo base:

- `yolov8n.pt` (Ultralytics)

Hiperparametros usados no treino inicial:

- `epochs=30`
- `imgsz=640`
- `batch=16`

### 2. Regra de Ameaca (THREAT)

Depois da deteccao, cada bounding box passa por uma regra heuristica baseada em:

- Posicao central na imagem.
- Area minima relativa da caixa.
- Posicao vertical.
- Razao de aspecto (orientacao do celular).

Se os criterios forem atendidos, a deteccao e marcada como `THREAT`; caso contrario, permanece como `mobile_phone`.

### 3. Inferencia com Acao de Seguranca

Para frames classificados com ameaca:

- Aplica obfuscacao visual na tela.
- Sobrepoe alerta textual.
- Salva imagem de saida processada.

### 4. Avaliacao Robusta

Inclui:

- Metricas oficiais de validacao do Ultralytics:
  - mAP@50
  - mAP@50-95
  - Precision
  - Recall
- Sweep de threshold de confianca (IoU >= 0.5).
- Escolha automatica de threshold otimo por F1.

### 5. Deploy em Tempo Real

Para video/webcam:

- Filtro temporal com histerese para reduzir flicker.
- Janela deslizante de decisoes por frame.
- Trigger e release com limiares distintos.
- Registro de evidencias e eventos em CSV.

### 6. Melhoria Iterativa

- Coleta de hard negatives para retreino.
- Exportacao para ONNX.
- Benchmark de latencia media, p95 e FPS estimado.

## Como Executar

### Opcao 1 - Google Colab (recomendado)

1. Abra o notebook principal no Colab.
2. Envie os arquivos ZIP do dataset para `/content`.
3. Execute as celulas em ordem.
4. Acompanhe os artefatos gerados em `/content`:
	- Dataset YOLO
	- Runs de treinamento
	- Resultados de inferencia
	- Evidencias e logs

### Opcao 2 - Ambiente local (Jupyter)

Pre-requisitos:

- Python 3.9+
- pip atualizado

Instale dependencias principais:

```bash
pip install ultralytics opencv-python numpy pandas matplotlib seaborn scikit-learn onnx onnxruntime
```

Observacoes:

- Ajuste os caminhos do notebook (ex.: `/content/...`) para diretarios locais.
- Para webcam, confirme permissao de camera e backend OpenCV no sistema.

## Saidas Geradas

Durante a execucao, o pipeline gera:

- Dataset convertido para YOLO.
- Pesos treinados (`best.pt`).
- Graficos e metricas de validacao.
- Imagens de validacao anotadas como SAFE/THREAT.
- Video de saida com alerta temporal.
- Capturas de evidencia em eventos de ameaca.
- CSV de eventos com frame, timestamp, score temporal e confianca.
- Modelo exportado para ONNX.

## Resultados Esperados

Com o notebook executado por completo, voce deve conseguir:

1. Detectar celulares em imagens e frames de video.
2. Diferenciar deteccao comum de situacao de risco via regra de negocio.
3. Obter metricas quantitativas para comparar versoes do modelo.
4. Preparar o modelo para uso em cenarios mais proximos de producao.

## Limitacoes

- A regra de ameaca e heuristica e sensivel ao enquadramento da cena.
- Resultados dependem da qualidade e diversidade do dataset.
- Webcam em notebook remoto (ex.: Colab) pode exigir adaptacoes.
- Um unico detector (`mobile_phone`) pode nao capturar contexto de uso sem classes auxiliares.

## Proximos Passos

- Expandir dataset com mais variacao de angulos, iluminacao e oclusoes.
- Ajustar hiperparametros e backbone para ganho de mAP.
- Incluir calibracao de threshold por ambiente (sala, laboratorio, prova).
- Integrar rastreamento (tracking) para maior estabilidade temporal.
- Empacotar como servico (API) para integracao com sistemas externos.

## Autoria

Trabalho desenvolvido para a disciplina de Visao Computacional (AP1), com foco em deteccao de telefones e classificacao de risco em cenarios de monitoramento.
