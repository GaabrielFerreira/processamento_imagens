# Lab 04 — Transformações Geométricas (cont.)
**Data:** 25/03/2026

Continuação das transformações afins com foco em **rotação** e **cisalhamento**, aplicadas com `cv.warpAffine`.

---

## Conceito: Rotação

A rotação gira os pixels de um plano em relação a uma origem. A matriz de afinidade para rotação em torno de (0,0) é:

```
| cos θ  -sin θ  0 |
| sin θ   cos θ  0 |
```

> Como o eixo y é positivo para baixo, θ positivo gira no sentido horário (CW). `getRotationMatrix2D` corrige automaticamente esse comportamento, retornando CCW para ângulo positivo.

---

## Exercício 1 — Rotação positiva e negativa

Carrega `butterfly.jpg` em RGB e aplica a função `rotate(img, angle)` que:

1. Converte o ângulo de graus para radianos com `math.radians`
2. Monta a matriz 2×3 manualmente com `np.float32`
3. Aplica com `cv.warpAffine(img, mat, (C, L))`

Subplot 1×3: original, rotação 30° (CW) e -60° (CCW).

> A imagem fica cortada pois a rotação é centrada em (0,0) — canto superior esquerdo.

---

## Exercício 2 — Rotações especiais (90°, 180°, 270°)

Ângulos múltiplos de 90° têm matrizes bem definidas:

| Ângulo | Matriz |
|--------|--------|
| 90°    | `[[0, -1], [1, 0]]` |
| 180°   | `[[-1, 0], [0, -1]]` |
| 270°   | `[[0, 1], [-1, 0]]` |

**Três formas de fazer o mesmo:**

1. **Matriz manual** com `np.float32` — rotação em torno de (0,0), imagem sai da área visível
2. **`getRotationMatrix2D(center, angle, scale)`** — rotação em torno do centro da imagem, resultado correto
3. **`cv.rotate(img, flag)`** — função otimizada do OpenCV com flags `ROTATE_90_CLOCKWISE`, `ROTATE_180` e `ROTATE_90_COUNTERCLOCKWISE`

---

## Exercício 3 — Rotação fora da origem

A rotação em torno de um ponto arbitrário `(Cx, Cy)` exige incluir uma translação na matriz:

```
tx = Cx·(1 - cos θ) + Cy·sin θ
ty = Cy·(1 - cos θ) - Cx·sin θ
```

**Passos:**
1. Carrega `apple.jpg` (imagem quadrada)
2. Calcula o centro `(C//2, L//2)`
3. Monta a matriz manualmente para **-45°** (negativo porque o eixo y é invertido)
4. Compara com `getRotationMatrix2D(center, 45, 1.0)` — os valores devem ser iguais (exceto erros de arredondamento)
5. Subplot 1×2: original e imagem rotacionada 45° em torno do centro

---

## Exercício 4 — Cisalhamento simples

Cisalhamento desloca linhas ou colunas linearmente:

```
x = u + Sx·v
y = v + Sy·u
```

Matriz afim:
```
| 1   Sx  0 |
| Sy  1   0 |
```

Função `shear(img, sx, sy)` monta a matriz e aplica `warpAffine`.

Carrega `left.jpg` e mostra subplot 1×4:
- Original
- Cisalhamento horizontal `sx=-0.1`
- Cisalhamento vertical `sy=0.1`
- Combinação `sx=-0.1` e `sy=0.1`

---

## Desafio — Cubo

Cria 3 quadrados 256×256 coloridos e combina em canvas 512×512 simulando as faces de um cubo.

**Face direita (vermelha):** cisalhamento vertical positivo (`sy=0.5`) + translação para direita

**Face esquerda (verde):** cisalhamento vertical negativo (`sy=-0.5`) + translação para baixo-esquerda

**Face topo (azul):** `getRotationMatrix2D` com escala > 1.0 e rotação 45°, posicionada no topo

**Combinação:**
```python
cube = red2 + green2 + blue2
```
As três imagens são somadas pixel a pixel sobre o canvas preto.
