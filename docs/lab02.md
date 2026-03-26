# Aula Lab. 2 — Amostragem, Quantização e Representação
**Data:** 11/03/2026

## Conceitos abordados
- Amostragem espacial (resolução)
- Quantização (profundidade de cor)
- Espaços de cor: RGB, YUV, HLS

---

## Exercício 1 — Diminuindo a Resolução (Amostragem)

**Conceito:** ao descartar pixels, reduzimos a resolução espacial. Como a informação descartada não pode ser recuperada, há **perda irreversível**.

O operador `[::2, ::2]` no Numpy seleciona 1 em cada 2 elementos (passo 2):

```python
baboon       = cv.imread(..., cv.IMREAD_GRAYSCALE)  # 512x512
baboon_256   = baboon[::2, ::2]                     # 256x256
baboon_128   = baboon_256[::2, ::2]                 # 128x128
baboon_64    = baboon_128[::2, ::2]                 # 64x64
```

O Matplotlib exibe cada subplot no mesmo tamanho físico → os pixels das imagens menores ficam maiores visualmente (interpolação por vizinho mais próximo).

---

## Exercício 2 — Desenhando uma Circunferência

**Conceito:** demonstra amostragem angular. Se não houver amostras suficientes, a circunferência aparece com buracos (undersampling).

### Setup da imagem
```python
M   = 64
img = np.zeros((M, M), dtype=np.uint8)  # imagem preta MxM
R   = M // 2
Cx  = M // 2 - 1   # centro (índice -1 == M-1 em Python, sem erro de borda)
Cy  = M // 2 - 1
```

### Loop de desenho
```python
n_samples = 4 * M * M   # Nyquist 2D: dobro do dobro da resolução
for angle in np.linspace(0, 360, n_samples):
    rad = np.deg2rad(angle)
    x = int(round(np.cos(rad) * R)) + Cx
    y = int(round(np.sin(rad) * R)) + Cy
    img[y, x] = 255
```

**Por que `n_samples = 4 * M * M`?** Nyquist diz que precisamos de pelo menos 2× a frequência máxima. Em 2D a circunferência de raio R tem `2πR ≈ πM` pixels de perímetro, então `4M²` amostras garante cobertura.

Aumentar M para 128, 256 e 512 evidencia como mais resolução = círculo mais suave.

---

## Exercício 3 — Quantização / Profundidade de Cor

**Conceito:** reduzir a quantidade de bits por canal gera o efeito **banding** (faixas visíveis em áreas de degradê).

### 8bpp → 4bpp
```python
img_4bpp_dark  = (img_8bpp / 16).astype(np.uint8)   # divide → escuro
img_4bpp_clear = (img_4bpp_dark * 16).astype(np.uint8)  # reescala → claro
```

| Operação | Efeito |
|----------|--------|
| `/16` | descarta os 4 bits baixos, imagem escura |
| `*16` | reescala de volta para 0–255, banding visível |

### 8bpp → 2bpp com máscara de bits
Mais eficiente: aplica `AND` para zerar os 6 bits baixos diretamente:

```python
mask    = np.array([0b11000000, 0b11000000, 0b11000000], dtype=np.uint8)
img_2bpp = img_8bpp & mask
```

Cada componente fica com apenas 4 valores possíveis (0, 64, 128, 192) → banding muito mais pronunciado.

---

## Exercício 4 — Representação de Cores (RGB, YUV, HLS)

**Conceito:** diferentes espaços de cor separam luminância de crominância de formas distintas.

```python
graf_yuv = cv.cvtColor(graf_bgr, cv.COLOR_BGR2YUV)
graf_hls = cv.cvtColor(graf_bgr, cv.COLOR_BGR2HLS)
```

### Comparação dos canais

| Espaço | Canal | Significado |
|--------|-------|-------------|
| RGB | R, G, B | vermelho, verde, azul |
| YUV | **Y** | luminância (brilho) |
| YUV | U, V | diferença de cor (crominância) |
| HLS | **H** | matiz (hue) — qual cor |
| HLS | **L** | luminosidade |
| HLS | **S** | saturação — quão "colorido" |

**Observações importantes:**
- Y (YUV) ≈ L (HLS): ambos representam luminância
- S preta = sem cor (cinza) → `H` é irrelevante nesses pontos
- G e B do RGB são similares por terem sensibilidade espectral parecida

### Rotação de Hue (bits)
Rotacionar os bits do canal H "desloca" todas as cores no círculo cromático:

```python
hls_shifted = graf_hls.copy()
hls_shifted[:, :, 0] = np.vectorize(rotl)(graf_hls[:, :, 0])
graf_shifted_rgb = cv.cvtColor(hls_shifted, cv.COLOR_HLS2RGB)
```

Resultado: vermelhos → amarelos, laranjas → verdes, azuis → rosas.
