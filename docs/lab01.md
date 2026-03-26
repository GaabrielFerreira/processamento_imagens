# Aula Lab. 1 — OpenCV: Fundamentos de Imagens
**Data:** 04/03/2026

## Conceitos abordados
- Leitura e exibição de imagens com OpenCV + Matplotlib
- Canais de cor (BGR vs RGB)
- Recorte de regiões (ROI)
- Conversão para tons de cinza

---

## Exercício 1 — Visualizando cada canal de cor

Uma imagem colorida no OpenCV é um array Numpy de shape `(linhas, colunas, 3)`.
Os 3 planos estão em ordem **BGR** (não RGB):

| Índice | Canal |
|--------|-------|
| `[:,:,0]` | **B** — Blue |
| `[:,:,1]` | **G** — Green |
| `[:,:,2]` | **R** — Red |

Para extrair cada canal usamos slicing Numpy:

```python
img_b = img[:, :, 0]  # todas as linhas, todas as colunas, canal B
img_g = img[:, :, 1]  # canal G
img_r = img[:, :, 2]  # canal R
```

Para exibir canais individualmente com Matplotlib, usamos `cmap='gray'` porque cada canal é um array 2D de intensidade:

```python
ax = plt.subplot(1, 3, 1)
ax.imshow(img_r, cmap='gray')
ax.set_title('R')
```

---

## Exercício 2 — Região de Interesse (ROI)

Recortes em arrays Numpy usam a sintaxe `[inicio:fim, inicio:fim]`.
Para um recorte de 200x200 a partir da linha `l` e coluna `c`:

```python
sub = img[l:l+200, c:c+200]        # todos os canais (imagem colorida)
sub = img[l:l+200, c:c+200, 1]     # apenas canal verde (array 2D)
sub = img[l:l+200, c:c+200, 2]     # apenas canal vermelho (array 2D)
```

**Três recortes pedidos:**
- `sub1`: linha 50, coluna 150, 200×200, **todos os canais** → 3D
- `sub2`: linha 100, coluna 200, 200×200, **canal verde** → 2D
- `sub3`: linha 150, coluna 250, 200×200, **canal vermelho** → 2D

> Recortes Numpy são **cópias**, não afetam o array original.

---

## Exercício 3 — Conversão para Grayscale

### Passo 1 — BGR → RGB
O OpenCV carrega imagens em BGR. Para exibir corretamente no Matplotlib:
```python
img_rgb = cv.cvtColor(img, cv.COLOR_BGR2RGB)
```

### Passo 2 — Conversão interna do OpenCV
```python
img_gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
```
O OpenCV usa pesos perceptuais calibrados para a resposta humana à luminância.

### Passo 3 — Conversão manual com pesos ITU-R BT.709
A fórmula padrão é:
$$L = R \times 0.2127 + G \times 0.7152 + B \times 0.0722$$

```python
gray_conv = np.float32([0.2127, 0.7152, 0.0722])
img_gray2 = img_rgb * gray_conv      # multiplica pixel a pixel
img_gray2 = img_gray2[:,:,1]         # o resultado fica no plano G
img_gray2 = img_gray2.astype(np.uint8)
```

**Por que os valores diferem?** O OpenCV usa coeficientes ligeiramente diferentes internamente (BT.601 ponderado), daí os valores numéricos não serem idênticos.

---

## Exercício 4 — Composição (desafio)

Combinar metade esquerda de uma imagem com metade direita de outra:

```python
lena   = cv.imread(cv.samples.findFile("lena.jpg"))
baboon = cv.imread(cv.samples.findFile("baboon.jpg"))

L, C, P = lena.shape
composition = np.zeros((L, C, P), dtype=np.uint8)
composition[:, :C//2, :] = baboon[:, :C//2, :]   # metade esq = baboon
composition[:, C//2:, :] = lena[:, C//2:, :]      # metade dir = lena
```

**Conceito chave:** slicing `[:, :C//2, :]` seleciona todas as linhas, colunas de 0 até C/2, e todos os canais.
