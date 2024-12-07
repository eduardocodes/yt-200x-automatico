# 🎥 YouTube 200% Automático

#### Vídeo: https://youtu.be/QcZJbSzaN_I

## URLs

Gerar imagem Leonardo AI - https://cloud.leonardo.ai/api/rest/v1/generations<br>
Buscar imagem Leonardo AI - https://cloud.leonardo.ai/api/rest/v1/generations/[INSERIR_GENERATION_ID]<br>
Gerar vídeo JSON2Video - https://api.json2video.com/v2/movies<br>
Gerar áudio Elevenlabs - https://api.elevenlabs.io/v1/text-to-speech/[INSERIR_VOICE_ID]<br>

## 👨‍💻 Códigos

## Criar tabelas Supabase

```bash
-- Criação da tabela "scenes"
CREATE TABLE scenes (
    id BIGINT NOT NULL,
    scene TEXT NOT NULL,
    ai_image_prompt TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NULL,
    image_url TEXT NULL,
    generation_id TEXT NULL,
    PRIMARY KEY (id)
);

-- Criação da tabela "stories"
CREATE TABLE stories (
    id BIGINT NOT NULL,
    script TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NULL,
    complete_video TEXT NULL,
    prompt TEXT NOT NULL,
    "aspect ratio" VARCHAR NOT NULL,
    width INTEGER NOT NULL,
    height INTEGER NOT NULL,
    style VARCHAR NOT NULL,
    leonardo_width INTEGER NULL,
    leonardo_height INTEGER NULL,
    PRIMARY KEY (id)
);

-- Criação da tabela "video_jobs"
CREATE TABLE video_jobs (
    id INTEGER NOT NULL,
    project_id VARCHAR NOT NULL,
    video_url VARCHAR NULL,
    audio_url VARCHAR NULL,
    merged_url VARCHAR NULL,
    created_at TIMESTAMP WITH TIME ZONE NULL,
    updated_at TIMESTAMP WITH TIME ZONE NULL,
    project_id_merged VARCHAR NULL,
    PRIMARY KEY (id)
);
```

### Definir dimensões e estilo

```bash
// Obter os dados de entrada do nó anterior
const inputData = items[0]?.json;

// Capturar os valores dos parâmetros do formulário
const aspectRatio = inputData?.["Aspect Ratio"];
const estilo = inputData?.["Estilo"];

// Verificar se os parâmetros obrigatórios foram enviados
if (!aspectRatio) {
    return [
        {
            json: {
                error: "O parâmetro 'Aspect Ratio' não foi encontrado nos dados de entrada.",
            },
        },
    ];
}

if (!estilo) {
    return [
        {
            json: {
                error: "O parâmetro 'Estilo' não foi encontrado nos dados de entrada.",
            },
        },
    ];
}

// Mapear estilos para seus respectivos valores
const estilosMap = {
    anime: "e71a1c2f-4f80-4800-934f-2c68979d8cc8",
    realistic: "b24e16ff-06e3-43eb-8d33-4416c2d75876",
    cinematic: "aa77f04e-3eec-4034-9c07-d0f619684628",
    "3d": "d69c8273-6b17-4a30-a13e-d6637ae1c644",
};

// Verificar se o estilo é válido
const estiloId = estilosMap[estilo.toLowerCase()];
if (!estiloId) {
    return [
        {
            json: {
                error: `Estilo não suportado: ${estilo}`,
            },
        },
    ];
}

// Variáveis para armazenar os valores de dimensões
let width, height, leonardo_width, leonardo_height;

// Processar os valores com base no Aspect Ratio
if (aspectRatio === "1920x1080") {
    width = 1920;
    height = 1080;
    leonardo_width = 1024;
    leonardo_height = 576;
} else if (aspectRatio === "1080x1920") {
    width = 1080;
    height = 1920;
    leonardo_width = 576;
    leonardo_height = 1024;
} else {
    return [
        {
            json: {
                error: `Aspect Ratio não suportado: ${aspectRatio}`,
            },
        },
    ];
}

// Retornar os valores calculados
return [
    {
        json: {
            aspectRatio: aspectRatio,
            width: width,
            height: height,
            leonardo_width: leonardo_width,
            leonardo_height: leonardo_height,
            modelId: estiloId,
        },
    },
];
```

### Wait

```bash
// Recupera o valor da variável "Wait" do node "Edit Fields"
const time = $node["Edit Fields"].json["Wait"];

// Converte o tempo de segundos para milissegundos
const waitTime = time * 1000;

// Função para aguardar o tempo definido
async function wait(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Aguarda o tempo configurado
await wait(waitTime);

// Retorna os dados de entrada sem alterações
return items;
```

### Preparar input

```bash
let processedLinks = new Set(); // Para evitar duplicação de imagens
let currentScenes = []; // Armazena as cenas do vídeo atual
let videos = []; // Armazena os vídeos finais
let sceneCount = 1; // Contador para as cenas
const maxScenesPerVideo = 3; // Número máximo de cenas por vídeo

// Obter largura e altura válidas do nó "Definir dimensões e estilos"
const dimensoes = $node["Definir dimensões e estilo"].json; // Referência ao nó "Definir dimensões e estilos"
const larguraOutput = dimensoes["width"];
const alturaOutput = dimensoes["height"];

// Validação dos valores de largura e altura
if (!larguraOutput || !alturaOutput) {
  throw new Error("Dimensões width e height não foram encontradas no nó 'Definir dimensões e estilos'.");
}

// Processar os itens para construir as cenas
items.forEach((item, index) => {
  const id = item.json["id"];
  const imagem = item.json["image_url"]; // URL da imagem

  // Verificar se ID e imagem estão presentes
  if (id && imagem) {
    if (!processedLinks.has(imagem)) {
      processedLinks.add(imagem);

      // Criar uma cena com a imagem atual
      const newScene = {
        comment: `Scene #${sceneCount++}`,
        elements: [
          {
            id: id,
            type: "image",
            src: imagem,
            duration: 5, // Duração padrão (5 segundos por imagem)
            zoom: 2, // Zoom padrão
            width: larguraOutput,
            height: alturaOutput,
            position: "center-center" // Posição padrão
          }
        ]
      };

      // Adicionar a cena ao conjunto de cenas do vídeo atual
      currentScenes.push(newScene);

      // Verificar se completamos o número máximo de cenas por vídeo
      if (currentScenes.length === maxScenesPerVideo || index === items.length - 1) {
        // Criar o vídeo com as cenas atuais
        const videoJson = {
          resolution: "full-hd",
          quality: "high",
          scenes: [...currentScenes] // Adiciona todas as cenas ao vídeo
        };

        // Adicionar o vídeo ao array de vídeos
        videos.push({ json: videoJson });

        // Limpar as cenas para o próximo vídeo
        currentScenes = [];
      }
    }
  }
});

// Retornar os vídeos como múltiplos itens no array
return videos;
```

### Resgatar credencial

```bash
// Input data
const input = $input.all(); // Obtém todos os dados do input

// Filtra apenas o primeiro objeto com o campo "Roteiro"
const result = input.find(item => item.json?.Roteiro);

// Retorna o objeto encontrado
return {
    json: result.json
};
```

### Resgatar input

```bash
// Itera sobre todos os itens recebidos no fluxo
return $items("Preparar input").map(item => {
  return {
    json: item.json
  };
});
```

### Remover duplicatas

```bash
// Obter os dados de entrada
const items = $input.all();

// Criar um mapa para rastrear cenas únicas
const uniqueScenes = new Map();

const filteredItems = items.filter(item => {
    const scene = item.json.scene; // Campo usado para verificar duplicatas
    if (!uniqueScenes.has(scene)) {
        uniqueScenes.set(scene, true);
        return true; // Manter item
    }
    return false; // Remover duplicata
});

// Retornar os itens filtrados
return filteredItems;
```

### Filtrar última linha

```bash
// Recebe o array de itens
const items = $input.all();

// Inicializa o ID do último item com audio_url preenchido
let lastFilledItemId = null;

// Identifica o último item com audio_url preenchido
items.forEach(item => {
  if (item.json.audio_url && (!lastFilledItemId || item.json.id > lastFilledItemId)) {
    lastFilledItemId = item.json.id;
  }
});

// Se nenhum item tiver audio_url preenchido, assume que o próximo ID deve ser o menor ID da lista
if (lastFilledItemId === null) {
  lastFilledItemId = Math.min(...items.map(item => item.json.id)) - 1;
}

// Filtra o próximo item com audio_url vazio e ID imediatamente após o último preenchido
const filteredItems = items.filter(item => {
  return (
    (!item.json.audio_url || item.json.audio_url.trim() === "") && // Checa se o audio_url está vazio ou string vazia
    item.json.id === lastFilledItemId + 1 // Checa se é o próximo ID na sequência
  );
});

// Retorna apenas os itens filtrados
return filteredItems;
```

### Preparar input1

```bash
const items = $input.all(); // Obter todos os itens do input

// Obter largura e altura válidas do nó "Definir dimensões e estilos"
const dimensoes = $node["Definir dimensões e estilo"].json; // Referência ao nó "Definir dimensões e estilos"
const larguraOutput = dimensoes["width"];
const alturaOutput = dimensoes["height"];

// Validação dos valores de largura e altura
if (!larguraOutput || !alturaOutput) {
  throw new Error("Dimensões width e height não foram encontradas no nó 'Definir dimensões e estilos'.");
}

// Array para armazenar as requisições preparadas
const videoRequests = [];

// Iterar pelos itens do input
items.forEach((item, index) => {
  // Obter as URLs de vídeo e áudio
  const videoUrl = item.json["video_url"];
  const audioUrl = item.json["audio_url"];
  const projectId = item.json["project_id"];

  // Garantir que ambas as URLs existem antes de prosseguir
  if (videoUrl && audioUrl) {
    // Criar a cena com o vídeo
    const scenes = [
      {
        comment: `Scene #${index + 1}`, // Nome da cena
        elements: [
          {
            type: "video",
            src: videoUrl,
            duration: -2 // Duração automática baseada no vídeo
          }
        ]
      }
    ];

    // Criar o elemento de áudio
    const audioElements = [
      {
        type: "audio",
        src: audioUrl,
        duration: -1 // Duração automática baseada no áudio
      }
    ];

    // Criar o payload da requisição
    const requestPayload = {
      id: `Merge Video + Audio #${projectId}`, // ID único para identificação
      fps: 25, // Frames por segundo
      cache: false, // Cache desativado
      draft: false, // Indica que não é um rascunho
      width: larguraOutput, // Largura obtida do nó "Definir dimensões e estilos"
      height: alturaOutput, // Altura obtida do nó "Definir dimensões e estilos"
      scenes: scenes, // Adicionar a cena com o vídeo
      quality: "high", // Qualidade máxima
      elements: audioElements, // Adicionar o áudio como elemento
      settings: {}, // Configurações adicionais (vazio por enquanto)
      resolution: "custom" // Resolução customizada
    };

    // Adicionar o payload ao array de requisições
    videoRequests.push({ json: requestPayload });
  }
});

// Retornar as requisições preparadas
return videoRequests;
```

### Preparar input2

```bash
let elementId = 1; // ID incremental para os elementos de vídeo
const processedLinks = new Set(); // Para evitar duplicação de vídeos
let elements = []; // Armazena os elementos de vídeo

// Obter largura e altura válidas do nó "Definir dimensões e estilos"
const dimensoes = $node["Definir dimensões e estilo"].json; // Referência ao nó "Definir dimensões e estilos"
const validLarguraOutput = dimensoes["width"];
const validAlturaOutput = dimensoes["height"];

// Validação dos valores de largura e altura
if (!validLarguraOutput || !validAlturaOutput) {
  throw new Error("Dimensões width e height não foram encontradas no nó 'Definir dimensões e estilos'.");
}

// Iterar pelos itens para processar as merged_url
items.forEach(item => {
  const id = item.json["id"];
  const mergedUrl = item.json["merged_url"]; // URL do vídeo já mesclado (merged_url)

  if (id && mergedUrl) {
    if (!processedLinks.has(mergedUrl)) {
      processedLinks.add(mergedUrl);
      elements.push({
        id: elementId++, // Incrementar o ID único do elemento
        type: "video", // Tipo: vídeo
        src: mergedUrl, // URL do vídeo
        duration: -1, // Duração automática baseada no vídeo
        zoom: 0, // Sem zoom
        width: validLarguraOutput, // Largura obtida do nó "Definir dimensões e estilos"
        height: validAlturaOutput, // Altura obtida do nó "Definir dimensões e estilos"
        position: "center-center" // Posição padrão (centralizado)
      });
    }
  }
});

// Criar as cenas com os elementos de vídeo
let scenes = elements.map(element => ({
  elements: [element] // Cada elemento é uma cena individual
}));

// JSON final para a API json2video
const outputJson = {
  id: "Vídeo Completo", // Identificador do projeto
  fps: 25, // Frames por segundo
  cache: false, // Cache desativado
  draft: false, // Indica que não é um rascunho
  width: validLarguraOutput, // Largura obtida do nó "Definir dimensões e estilos"
  height: validAlturaOutput, // Altura obtida do nó "Definir dimensões e estilos"
  scenes: scenes, // Todas as cenas geradas a partir dos elementos
  quality: "high", // Qualidade alta
  elements: [
    {
      type: "subtitles", // Tipo: legendas
      settings: {
        "all-caps": true, // Texto em caixa alta
        position: "mid-bottom-center", // Posição das legendas
        "font-size": 75, // Tamanho da fonte
        "font-family": "Luckiest Guy", // Fonte usada
        "outline-width": 5 // Largura do contorno
      }
    }
  ],
  settings: {}, // Configurações adicionais vazias
  resolution: "custom" // Resolução customizada
};

// Retornar o JSON final como um array com um único item
return [{ json: outputJson }];
```

### Preparar input3

```bash
let elementId = 1; // ID incremental para os elementos de vídeo
const processedLinks = new Set(); // Para evitar duplicação de vídeos
let elements = []; // Armazena os elementos de vídeo

// Obter largura e altura válidas do nó "Definir dimensões e estilos"
const dimensoes = $node["Definir dimensões e estilo"].json; // Referência ao nó "Definir dimensões e estilos"
const validLarguraOutput = dimensoes["width"];
const validAlturaOutput = dimensoes["height"];

// Validação dos valores de largura e altura
if (!validLarguraOutput || !validAlturaOutput) {
  throw new Error("Dimensões width e height não foram encontradas no nó 'Definir dimensões e estilos'.");
}

// Iterar pelos itens para processar as merged_url
items.forEach(item => {
  const id = item.json["id"];
  const mergedUrl = item.json["merged_url"]; // URL do vídeo já mesclado (merged_url)

  if (id && mergedUrl) {
    if (!processedLinks.has(mergedUrl)) {
      processedLinks.add(mergedUrl);
      elements.push({
        id: elementId++, // Incrementar o ID único do elemento
        type: "video", // Tipo: vídeo
        src: mergedUrl, // URL do vídeo
        duration: -1, // Duração automática baseada no vídeo
        zoom: 0, // Sem zoom
        width: validLarguraOutput, // Largura obtida do nó "Definir dimensões e estilos"
        height: validAlturaOutput, // Altura obtida do nó "Definir dimensões e estilos"
        position: "center-center" // Posição padrão (centralizado)
      });
    }
  }
});

// Criar as cenas com os elementos de vídeo
let scenes = elements.map(element => ({
  elements: [element] // Cada elemento é uma cena individual
}));

// JSON final para a API json2video
const outputJson = {
  id: "Vídeo Completo", // Identificador do projeto
  fps: 25, // Frames por segundo
  cache: false, // Cache desativado
  draft: false, // Indica que não é um rascunho
  width: validLarguraOutput, // Largura obtida do nó "Definir dimensões e estilos"
  height: validAlturaOutput, // Altura obtida do nó "Definir dimensões e estilos"
  scenes: scenes, // Todas as cenas geradas a partir dos elementos
  quality: "high", // Qualidade alta
  elements: [],
  settings: {}, // Configurações adicionais vazias
  resolution: "custom" // Resolução customizada
};

// Retornar o JSON final como um array com um único item
return [{ json: outputJson }];
```

### Resgatar credencial1

```bash
// Capture o input do node "Resgatar credencial"
const input = $node["Resgatar credencial"]?.json || {};

// Retorne apenas o objeto dentro de `input`
return {
    json: input,
};
```

---

Made with 💙 by eduardocodes 👋
