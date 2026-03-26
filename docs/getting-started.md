# Getting Started — Processamento de Imagens com OpenCV

## Sobre o projeto

Exercícios práticos de processamento de imagens usando **Python + OpenCV**, organizados por aula.
Cada aula é um **Jupyter Notebook** (arquivo `.ipynb`) com teoria em texto e células de código para implementar.

---

## Estrutura do projeto

```
processamento_imagens/
├── data/          → imagens utilizadas nos exercícios
├── labs/          → notebooks com os exercícios por aula
│   ├── lab01.ipynb   (04/03/2026 - Fundamentos: canais, ROI, grayscale)
│   ├── lab02.ipynb   (11/03/2026 - Amostragem, quantização, espaços de cor)
│   ├── lab03.ipynb   (18/03/2026 - Transformações geométricas)
│   └── lab04.ipynb   (25/03/2026 - Rotação e cisalhamento)
└── docs/          → documentação explicativa de cada aula
    ├── lab01.md
    ├── lab02.md
    ├── lab03.md
    └── getting-started.md
```

---

## Pré-requisitos

- Python 3.12+
- O projeto inclui um ambiente virtual (`.venv`) com todas as dependências já instaladas

### Dependências
| Biblioteca | Versão | Uso |
|------------|--------|-----|
| `opencv-python` | 4.13 | processamento de imagens |
| `numpy` | 2.4 | manipulação de arrays e matrizes |
| `matplotlib` | 3.10 | visualização de imagens e gráficos |
| `ipykernel` | 7.2 | integração Python com Jupyter |

---

## Como executar os notebooks

### VS Code (recomendado)

1. Abra o VS Code na raiz do projeto (`processamento_imagens/`)
2. Abra qualquer arquivo `labs/lab0X.ipynb`
3. No canto superior direito, clique em **"Select Kernel"**
4. Selecione **".venv (Python 3.12)"**
5. Execute as células com **`Shift + Enter`**

### Terminal

```bash
source .venv/bin/activate
jupyter notebook labs/lab01.ipynb
```

---

## Executando células

Um notebook é dividido em **células**:

- **Markdown** → texto explicativo, não precisa executar
- **Code** → código Python, execute em ordem de cima para baixo

| Atalho | Ação |
|--------|------|
| `Shift + Enter` | executa a célula e avança para a próxima |
| `Ctrl + Enter` | executa a célula e permanece nela |

> As células têm dependências entre si. Sempre execute na ordem para evitar erros.

---

## Problemas comuns

**`ModuleNotFoundError: No module named 'cv2'`**
→ O kernel selecionado não é o `.venv`. Troque pelo ambiente correto no canto superior direito.

**Erro no `findFile` / imagem não carrega**
→ Certifique-se de que o notebook foi aberto a partir da raiz do projeto, não de dentro de `labs/`.
A célula `cv.samples.addSamplesDataSearchPath("./data")` deve ser executada antes de qualquer `imread`.

**Nenhuma imagem aparece**
→ Verifique se `plt.show()` está presente ao final da célula. Sem ele, o gráfico não é renderizado.
