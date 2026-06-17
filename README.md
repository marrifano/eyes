# Eyes — Protótipo WebAR

Experiência de realidade aumentada que roda no **navegador do celular**.
Escaneia um QR → abre a página → aponta a câmera para o marcador → os
personagens (feitos no Blender) aparecem **em volta** dele.

Stack: **A-Frame + AR.js** (puro HTML/JS, sem instalar app).

---

## 1. Exportar os personagens do Blender (o passo principal)

A cena lê arquivos **`.glb`** (glTF binário) — é o formato ideal pra web:
um único arquivo que já leva malha, materiais, texturas e animação juntos.

### Passo a passo no Blender

1. Selecione o personagem (objeto + armature, se tiver animação).
2. Menu **File → Export → glTF 2.0 (.glb/.gltf)**.
3. Na janela de exportação, configure:
   - **Format:** `glTF Binary (.glb)`  ← um arquivo só, mais fácil pra web
   - **Include → Selected Objects:** marque (exporta só o que selecionou)
   - **Transform → +Y Up:** **marcado** (essencial — A-Frame/three.js usam Y pra cima)
   - **Geometry → Apply Modifiers:** marcado
   - **Geometry → Materials:** `Export`
   - Se tiver animação: aba **Animation** marcada (Animation + Skinning).
4. Salve como `personagem1.glb` dentro da pasta `models/`.
5. Repita para os outros (`personagem2.glb`, `personagem3.glb`).

### Dicas pra ficar leve e rodar bem no celular
- **Escala/tamanho:** modele numa escala razoável. Se ficar gigante ou minúsculo
  na AR, ajuste o atributo `scale` no `index.html` (ex.: `scale="0.3 0.3 0.3"`).
- **Origem (pivot):** no Blender, posicione a origem do objeto na base/pés
  (`Object → Set Origin → Origin to 3D Cursor` com o cursor no chão). Assim ele
  "pisa" no marcador em vez de afundar/flutuar.
- **Polígonos:** mire em poucos milhares de tris por personagem. Use o modifier
  **Decimate** se estiver muito pesado.
- **Texturas:** 1024px costuma bastar. Texturas 4K deixam o carregamento lento no 4G.
- **Tamanho do arquivo:** tente manter cada `.glb` abaixo de ~3–5 MB.

> Animação: se exportou com animação, ela toca sozinha por causa do
> atributo `animation-mixer` já presente em cada personagem no `index.html`.

---

## 2. Como os modelos entram na cena

No `index.html`, cada personagem é um `<a-entity>` dentro do `<a-marker>`:

```html
<a-entity
  gltf-model="#personagem1"   <!-- referencia o asset carregado -->
  position="0 0 1"            <!-- POSIÇÃO em volta do marcador (metros) -->
  rotation="0 180 0"         <!-- pra qual lado ele "olha" -->
  scale="0.5 0.5 0.5"        <!-- tamanho -->
  animation-mixer>           <!-- toca a animação do Blender -->
</a-entity>
```

**Eixos (relativos ao marcador):**
- `x`: esquerda (−) / direita (+)
- `y`: baixo (−) / cima (+)
- `z`: atrás (−) / frente, em direção a você (+)

Para distribuir mais personagens **em círculo** ao redor, é só variar `x` e `z`.
Ex.: 4 personagens nos lados → `0 0 1`, `1 0 0`, `0 0 -1`, `-1 0 0`.

### Testar sem modelo (antes de exportar do Blender)
Quer ver a câmera/AR funcionando já? Troque um `<a-entity gltf-model=...>`
por uma forma simples, ex.:

```html
<a-box position="0 0.5 0" color="tomato" scale="0.5 0.5 0.5"></a-box>
```

Aí o cubo aparece sobre o marcador sem precisar de nenhum `.glb`.

---

## 3. Marcador "Hiro" (pra testar agora)

O protótipo usa o marcador padrão **Hiro**. Abra esta imagem em OUTRA tela
(monitor/segundo celular) e aponte a câmera pra ela:

  https://raw.githgithack.com/AR-js-org/AR.js/master/data/images/hiro.png

(imprimir também funciona). Depois dá pra trocar por um marcador com a arte
do seu projeto — me avisa que eu te mostro como gerar.

---

## 4. Publicar (HTTPS obrigatório pra câmera)

O navegador **só libera a câmera em HTTPS**. A forma grátis mais fácil é o
**GitHub Pages**:

1. Crie um repositório no GitHub e suba estes arquivos (`index.html`, `models/`).
2. No repo: **Settings → Pages → Source: Deploy from a branch → main / root**.
3. Em ~1 min ele te dá uma URL `https://seu-usuario.github.io/eyes/`.

### Testar local rápido (no PC)
```bash
# Python já instalado? Na pasta do projeto:
python -m http.server 8000
# abre http://localhost:8000  (localhost conta como seguro p/ câmera)
```
Para abrir no celular pela rede local sem HTTPS o navegador bloqueia a câmera —
por isso o jeito prático de testar no celular é publicar no GitHub Pages.

---

## 5. Gerar o QR code

Depois de ter a URL pública (ex.: `https://seu-usuario.github.io/eyes/`),
gere um QR apontando pra ela. Qualquer gerador serve
(ex.: site qr-code-generator, ou eu te gero um script). A pessoa escaneia →
abre a página → aponta pro marcador → vê os personagens. 🎉

---

## Limitação honesta (pra você saber)
Mapeamento 3D completo da sala (SLAM real, oclusão, detectar o chão) é
limitado no WebAR grátis. Aqui os personagens aparecem **ancorados ao redor
do marcador**, o que já entrega a sensação de "personagens em volta".
Se mais pra frente quiser o mapeamento completo do ambiente, os caminhos são
**8thWall** (WebAR pago) ou **Unity + AR Foundation** (app nativo) — e os mesmos
`.glb` do Blender são reaproveitados.
