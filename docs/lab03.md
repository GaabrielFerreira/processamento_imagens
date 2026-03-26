# Aula Lab. 3 — Transformações Espaciais/Geométricas
**Data:** 18/03/2026

## Conceitos abordados
- Transformação reversa (forward vs inverse mapping)
- Interpolação (vizinho mais próximo)
- Matrizes de transformação afim
- Translação, escala e combinação de transformações

---

## Exercício 1 — Coordenadas e Espelhamento Horizontal

**Conceito:** transformações geométricas não alteram o valor dos pixels, apenas mudam sua posição. A abordagem padrão é a **transformação reversa**: para cada pixel de destino `(u, v)`, calcula-se o pixel de origem `(x, y)`.

### Espelhamento horizontal
- `y = v` (linha não muda)
- `x = (C - 1) - u` (coluna é invertida)

```python
fish = cv.imread(cv.samples.findFile('HappyFish.jpg'), cv.IMREAD_COLOR_RGB)
L, C, P = fish.shape
fish_mirrorh = np.zeros((L, C, P), dtype=np.uint8)

for v in range(L):
    for u in range(C):
        x = (C - 1) - u
        y = v
        fish_mirrorh[v, u, :] = fish[y, x, :]
```

> O mesmo resultado pode ser obtido com `fish[:, ::-1, :]` (slice reverso), mas o loop for é didático para entender a transformação reversa.

---

## Exercício 2 — Translação com `warpAffine`

**Conceito:** translação desloca todos os pixels por `(tx, ty)` pixels.

A matriz afim simplificada (2×3) para translação é:
$$M = \begin{bmatrix}1 & 0 & tx \\ 0 & 1 & ty\end{bmatrix}$$

```python
def transpose(img, tx, ty):
    L, C = img.shape[:2]
    M = np.float32([[1, 0, tx], [0, 1, ty]])
    return cv.warpAffine(img, M, (C, L))
```

### Variações
| Função | `dsize` | `borderMode` | Comportamento |
|--------|---------|--------------|---------------|
| `transpose` | `(C, L)` | padrão (preto) | área original, borda preta |
| `transpose2` | `(C*2, L*2)` | padrão | área dobrada, vê a imagem deslocada |
| `transpose3` | `(C*2, L*2)` | `BORDER_REPLICATE` | borda repete pixel da borda da imagem |

```python
# área dobrada com borda replicada
cv.warpAffine(img, M, (C*2, L*2), borderMode=cv.BORDER_REPLICATE)
```

---

## Exercício 3 — Escala com `warpAffine`

**Conceito:** escala altera o tamanho da imagem multiplicando as coordenadas.

A matriz afim para escala é:
$$M = \begin{bmatrix}sx & 0 & 0 \\ 0 & sy & 0\end{bmatrix}$$

```python
def scale(img, sx, sy=0):
    if sy == 0:
        sy = sx           # escala uniforme quando sy omitido
    L, C = img.shape[:2]
    M = np.float32([[sx, 0, 0], [0, sy, 0]])
    return cv.warpAffine(img, M, (C, L))
```

| Chamada | Resultado |
|---------|-----------|
| `scale(img, 0.75)` | 25% menor em ambos os eixos |
| `scale(img, 1, 2)` | dobro de altura, largura igual |
| `scale(img, 1.1, 0.8)` | +10% em X, -20% em Y |

---

## Exercício 4 — Combinação de Transformações (Desafio)

**Conceito:** matrizes 3×3 permitem combinar múltiplas transformações com multiplicação matricial.

```python
mat_t = np.float32([[1, 0, tx], [0, 1, ty], [0, 0, 1]])   # translação
mat_s = np.float32([[sx, 0, 0], [0, sy, 0], [0, 0, 1]])   # escala

mtx = mat_t @ mat_s  # ou np.matmul(mat_t, mat_s)
# resultado: [[sx, 0, tx], [0, sy, ty], [0, 0, 1]]

mtx = np.delete(mtx, 2, axis=0)  # remove 3ª linha para usar no warpAffine
return cv.warpAffine(img, mtx, (C*2, L*2))
```

**Por que multiplicar `mat_t @ mat_s` e não `mat_s @ mat_t`?**
A ordem importa: `T @ S` escala primeiro, depois translada. Isso centraliza a imagem escalada na área de destino.

A matriz resultante combina as duas operações em uma única passagem sobre a imagem → mais eficiente.
