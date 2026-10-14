# CHANGELOG — ReceitaPro 6.2

Data: 14/10/2026. Aplicação entregue: `ReceitaPro_6.2_FINAL.html`, arquivo único com CSS/JavaScript vanilla incorporados, sem dependências, CDN ou etapa de build adicionados. Schema de dados: **12**.

O cadastro foi recuperado; estoque, receita, produção e venda persistem após recarga. A causa reproduzida neste anexo foi o ID DOM ausente `initialStockField`, e não `persist()` async: essa função já era síncrona. Foram corrigidas também corridas e persistência da fila de nuvem, sem fazer o salvamento local esperar pela rede.

Os trechos ANTES/DEPOIS abaixo são reais, extraídos dos checkpoints da implementação; o HTML contém todas as etapas integradas. Quando o comportamento já existia, a seção informa a verificação e não inventa uma alteração. Os arquivos auxiliares citados nos relatórios são evidências de desenvolvimento; **não são necessários para usar o HTML**.

## Estado de cada item

| Item | Estado | Observação |
|---|---|---|
| 1.1 | aplicado | Funções e call sites localizados. |
| 1.2 | aplicado | Falha original reproduzida com stack; causa corrigida. |
| 1.3 | aplicado | Local síncrono, nuvem desacoplada e enfileiramento ordenado. |
| 1.4 | aplicado | Promises de nuvem tratadas com logs. |
| 1.5 | não aplicado / não necessário | `hydrateSyncBase` não existia; nenhuma remoção foi necessária. |
| 1.6 | aplicado / verificado | Nenhum await de nuvem ficou em persist/commit/render. |
| A1–A4 | aplicado / verificado | Comportamentos corretos já presentes foram preservados e testados. |
| B1–B6 | aplicado | Sidebar, largura, dialog redundante, fila, Maps e gradiente. |
| C1–C9 | aplicado | Foco, campos, ARIA, eventos, impressão, lembretes e auditoria. |


| 5.1 | aplicado | Atalhos adicionais |
| 5.2 | aplicado | Paleta com prefixos e teclado |
| 5.3 | aplicado | Enter para editar/contar nas linhas |
| 5.4 | aplicado | Breadcrumb contextual |
| 5.5 | aplicado | Badges de estoque/fiados/produtos |
| 5.6 | aplicado | FAB com pressão de 300 ms |
| 5.7 | aplicado | Combos nativos quando lista tem mais de 15 opções |
| 5.8 | aplicado | Excluir e desfazer ações reversíveis |
| 5.9 | aplicado | Busca visual dos históricos |
| 5.10 | aplicado | Ordenação cronológica/numérica |
| 5.11 | aplicado | Skeleton de nuvem após 300 ms |
| 5.12 | aplicado | Preferências locais por usuário |
| 5.13 | aplicado | Três toasts empilháveis |
| 5.14 | aplicado | Estado real de salvamento e retry |
| 5.15 | aplicado | Tema de alto contraste (opcional aplicado) |
| 5.16 | aplicado | Paginação de 50 em tabelas com mais de 100 registros |
| 5.17 | aplicado | Duplicar receita como variação |
| 5.18 | aplicado | Impressão das duas listas de compras |
| 5.19 | aplicado | Lembrete de backup de 7 dias |
| 5.20 | aplicado | Conferência rápida com contagem |



## Testes executados e limites

| Verificação | Resultado |
|---|---|
| Fluxo obrigatório a–j, interface real Chromium | passou; 690 g de Leite condensado, 8 produtos prontos e 5 eventos após recargas |
| Integração da Etapa 5 | 14/14 verificações; nenhum item marcado como não executado |
| Tabelas, combos, edição, prefs e mudança de dia | 9/9 cenários específicos passaram |
| Navegação, prefixos, cancelamento, FAB e contagem | 9/9 cenários específicos passaram |
| Feedback, status/erro/retry, toast e desfazer | 13/13 testes centrais passaram |
| Alto contraste, impressão e backup | 10/10 testes auxiliares e 11 casos de timestamp passaram |
| Fila de nuvem simulada | 9/9; local salva antes da liberação da nuvem, gzip/legados, offline, erro sem loop e commit durante upload |
| Cadastro durante abertura de workspace | passou; o item local permanece junto com o item remoto no IndexedDB |
| Transação IDB falha, fallback consegue salvar | passou; a leitura imediata e após reload usam o valor correto |
| Backups sintéticos 5.1/6.0/6.1/9/10/11 | 24/24 casos e 120 verificações; desktop/mobile, IndexedDB/fallback, import/reload/export/reimport |
| Bloco B | 12 verificações passaram; contraste AA, Maps, gradiente, sidebar 236/210 px e móvel |
| Bloco C | 14 verificações passaram; 28 inputmodes, ARIA/foco/print/lembretes/auditoria |
| Visual claro/escuro/alto contraste | 9 verificações em 1440 px/390 px, sem overflow; 6 capturas inspecionadas |
| Clipboard rejeitado | 3 caminhos passaram; log + alerta/toast de recuperação, sem rejeição órfã |
| Sintaxe do módulo JavaScript | `node --check` passou |

O fluxo normal terminou sem console.error, pageerror, rejeições não tratadas ou funções expostas órfãs. Os testes de falha deliberada de armazenamento registram o erro esperado e comprovam recuperação por retry; isso não é um erro inesperado escondido.

O último ajuste foi adicionar logs aos três fallbacks de clipboard. Cadastro, integração e clipboard foram executados novamente no SHA final. B/C, visual, compatibilidade e fila haviam passado antes desse ajuste exclusivo de logs; seus resultados foram conservados porque os respectivos comportamentos permaneceram iguais.

As fixtures de backup são sintéticas: nenhum backup histórico autêntico foi fornecido. Supabase foi simulado sem credenciais; autenticação/RPC de um projeto real **não foi validado**. Firefox e WebKit estão **skipped**, pois os executáveis não estão instalados. Os testes usaram HTTP local e service workers bloqueados: a política do Chromium desse ambiente impede `file://`. Nenhuma instrumentação de teste está no HTML entregue.

## Limitações e problemas conhecidos remanescentes

- A instalação PWA/offline por service worker não foi validada. O anexo referencia `sw.js`, `icon-192.png` e `icon-512.png`, mas esses arquivos não foram enviados. A integração existente foi preservada; o HTML único não substitui esses recursos na publicação HTTP(S).
- Filas antigas já corrompidas como `gzip:{}` pelo fallback original não têm bytes recuperáveis. O novo formato evita essa perda e a leitura inválida registra diagnóstico; recuperar dados previamente perdidos depende de backup válido.
- Ajustes antigos não registram saldo contado: a diferença da conferência é identificada como estimativa pelo histórico. Novos ajustes de contagem guardam `stockAfter`, permitindo referência precisa; saldo já igual à contagem continua sem gerar movimentação, conforme a regra anterior.
- Se IndexedDB e localStorage falharem simultaneamente, o app mostra erro e retry; nenhum browser pode prometer persistência sem armazenamento disponível.
- Nenhuma falha reproduzida ficou sem correção no escopo executado. As limitações acima não foram apresentadas como funcionalidades validadas.

## Adicionado, substituído e intocado

**Adicionado:** atalhos e prefixos, navegação de tabelas, breadcrumb/badges, radial FAB, combos/filtros/ordenação/colunas/paginação, prefs por usuário, skeleton, fila de toasts, retry, alto contraste, edição de perda/fechamento por teclado, impressão de compras, lembrete de backup, conferência e metadado de contagem.

**Substituído:** CSS duplicado da sidebar por tokens consistentes; arrays de ações por Maps; handlers complexos por eventos equivalentes; confirmação de exclusões reversíveis por desfazer seguro; toast único por pilha; estado de salvamento presumido por confirmação de escrita; duplicação pronta por formulário de variação; gzip não serializável por bytes compatíveis com JSON. O dialog de alerta removido não tinha referência de uso: alertas continuam pelo dialog ativo de `ui`.

**Intocado funcionalmente:** motor `Core` de custo/estoque/métricas, cálculos e snapshots históricos, carteira, pagamentos/fiados, fechamento diário, exportação CSV, backups e importações antigas, fluxos de autenticação/empresas/convites e integração PWA. Exclusões de entidades usadas continuam bloqueadas e apagar todos os dados continua exigindo confirmação. A fila de nuvem recebeu correções de robustez, sem retirar sincronização.

## Checkpoints por etapa

As etapas 1, 2, 3 e 4 só avançaram após seus gates. O fluxo a–j foi repetido depois de B/C e na entrega consolidada. Funções expostas e handlers equivalentes foram verificados; a inicialização abre sem erro inesperado. Contraste claro/escuro foi validado em B e três temas na entrega. Os códigos seguintes documentam cada item.


## Etapa 1 — correção do cadastro e independência local/nuvem

### Diagnóstico reproduzido

O arquivo fornecido já tinha `persist()` e `commit()` síncronos; a hipótese de propagação de `async` não explica a falha de inicialização deste anexo. A reprodução real em Chromium encontrou:

```text
TypeError: Cannot read properties of null (reading 'classList')
    at resetIngredientForm (http://127.0.0.1:39075/ReceitaPro.html:623:415)
    at init (http://127.0.0.1:39075/ReceitaPro.html:889:40)
```

Causa: `resetIngredientForm()` e `editIngredient()` referenciavam `initialStockField`, ID inexistente no DOM. Evidência completa: `stage1-original.json`.

### Itens e testes manuais

- **1.1 / 1.2 — auditoria e reprodução:** localizadas `persist`, `commit`, `enqueueCloudSnapshot`, `loadSyncBase`, `processCloudQueue`, `cloudSave`. `hydrateSyncBase` não existe e não há função exposta órfã associada. Teste: abrir o original e conferir o stack acima.
- **Correção efetiva do cadastro:** adicionado o wrapper interno `initialStockField` ao checkbox e saldo inicial no painel 3, mantendo `ingStepPanel3`, ARIA e wizard existentes. O wrapper é mostrado no cadastro e ocultado na edição, inclusive ao navegar novamente para a etapa 3. Teste: cadastrar insumo, editar, conferir que saldo inicial não pode ser alterado pela edição e cancelar para restaurar o fluxo de cadastro.
- **1.3 — persistência independente:** `persist()` e `commit()` continuam funções síncronas. A escrita local captura seu clone imediatamente; toda normalização/enfileiramento/processamento de nuvem fica em promises com `.catch`. O processamento aguarda a mesma tarefa de enfileiramento e verifica `cloud.busy` ao executar, evitando corrida e processamento antecipado. Teste: repetir cadastro em modo offline ou com RPC da nuvem rejeitado; retornar da operação imediatamente e confirmar os dados após recarregar.
- **1.4 — tratamento das promises:** os dois call sites de `enqueueCloudSnapshot` capturam rejeição explicitamente; gatilhos `online`, inicialização, polling e continuação da fila também capturam/logam rejeições. Alertas de falha local agora tratam sua promise. Teste: observar `unhandledrejection` durante enfileiramento, processamento e falha de rede simulada.
- **1.5 / 1.6 — async restrito à nuvem:** nenhuma função `hydrateSyncBase` precisou ser removida; todos os `await loadSyncBase()` continuam em funções async de nuvem, nenhum em persist/commit/render. Teste: checagem estática de call sites e `constructor.name` das duas funções síncronas.
- **Integridade offline:** enfileiramentos e gravações da fila são serializados; `enqueueCloudSnapshot` aguarda `saveCloudQueue`. Cada gravação captura um snapshot independente; remoção identifica o item pelo ID, aguarda armazenamento e restaura o item em memória se a gravação falhar, seguido de uma escrita de restauração com erro tratado para superar snapshots concorrentes já agendados. Após falha de rede, a fila fica pendente e aguarda um gatilho posterior, sem ciclo imediato infinito. Teste: falhar RPC e confirmar fila/IndexedDB após reload; recuperar rede e sincronizar; inserir vários commits durante gravação.
- **Fila comprimida compatível com fallback:** `gzipText` retorna array serializável de bytes; `gunzipText` aceita esse formato e também ArrayBuffer/typed array antigos. `payload.raw` e itens `data` v6.0 continuam aceitos. O IndexedDB mantém a compressão e o fallback localStorage preserva os bytes no JSON. Teste: forçar IndexedDB indisponível, enfileirar offline, recarregar e recuperar item; importar item gzip ArrayBuffer e fila raw/data antiga.
- **Proteção de retorno antigo:** RPCs usam o workspace do item; mudanças de empresa/logout abortam aplicação da resposta. Dados mesclados só substituem o banco local quando o snapshot local ainda equivale ao último item enviado e não existe novo enqueue pendente. Teste: atrasar RPC, registrar outro insumo/compra e soltar RPC; confirmar que ambos os dados permanecem.

### Validação

- `node --check` no script extraído: aprovado.
- `stage1-fixed.json`: passos a–j aprovados em Chromium/localhost com formulários reais e IndexedDB; cadastro de Leite condensado (R$ 7,50/395 g), compra de 2 embalagens (790 g), receita, produção (690 g restantes/10 unidades prontas), venda (8 prontas), reload em cada gate, zero erros vermelhos e zero rejeições órfãs.
- Testes complementares da nuvem: 9/9 passaram; provas adicionais de abertura concorrente e fallback transacional também passaram.
- `window.exposed`: nenhuma função removida/renomeada nesta etapa; nenhum handler removido sem equivalente.
- Sidebar: intocada nesta etapa; revisão de contraste pertence aos blocos seguintes.
- Schema e importação de backups: intocados nesta etapa.

### Diff real ANTES → DEPOIS

```diff
--- ReceitaPro_6.1_BLOCO_A.html
+++ ReceitaPro_6.2_FINAL.html
@@ -98,6 +98,6 @@
 </div><div class="cost-preview" id="ingCostPreview">Preencha preço e quantidade para ver o custo por unidade.</div></div>
 <div aria-labelledby="ingStepTab3" class="guided-block wizard-hidden" data-ing-step="3" id="ingStepPanel3" role="tabpanel"><div class="guided-title"><span>3</span><div><b>Você já tem esse produto?</b><small>Isso será estoque inicial, não compra</small></div></div>
-<label class="switch-row"><input id="ingAlreadyHave" onchange="toggleInitialStock()" type="checkbox"/><span><b>Sim, eu já tenho em casa/estoque</b><small>Marque para informar quanto realmente sobrou agora.</small></span></label>
-<label class="field hide" id="ingInitialWrap"><span>Quanto você tem agora</span><input class="input" id="ingInitial" min="0" step="any" type="number" value="0"/><div class="field-help">Ex.: embalagem original de 1000 g, mas você pesou e sobraram 680 g → coloque <b>680</b>.</div></label>
+<div id="initialStockField"><label class="switch-row"><input id="ingAlreadyHave" onchange="toggleInitialStock()" type="checkbox"/><span><b>Sim, eu já tenho em casa/estoque</b><small>Marque para informar quanto realmente sobrou agora.</small></span></label>
+<label class="field hide" id="ingInitialWrap"><span>Quanto você tem agora</span><input class="input" id="ingInitial" min="0" step="any" type="number" value="0"/><div class="field-help">Ex.: embalagem original de 1000 g, mas você pesou e sobraram 680 g → coloque <b>680</b>.</div></label></div>
 </div>
 <div aria-labelledby="ingStepTab4" class="guided-block wizard-hidden" data-ing-step="4" id="ingStepPanel4" role="tabpanel"><div class="guided-title"><span>4</span><div><b>Quando avisar?</b><small>Opcional</small></div></div><label class="field"><span>Estoque mínimo</span><input class="input" id="ingMin" min="0" step="any" type="number" value="0"/><div class="field-help">Quando o saldo chegar aqui, o ReceitaPro avisa que é hora de repor.</div></label></div>
@@ -470,6 +470,6 @@
 })();
 const AUTO_INDEX_KEY='receitaProAutoBackupIndex',AUTO_ITEM_PREFIX='receitaProAutoBackup:',CLOUD_QUEUE_KEY='receitaProCloudPendingQueue';
-let autoBackups=[],cloudPendingQueue=[],storageWriteChain=Promise.resolve(),cloudQueueWriteChain=Promise.resolve(),syncBaseCache=new Map();
-function enqueueStorageWrite(key,value){storageWriteChain=storageWriteChain.then(()=>storage.set(key,deepClone(value))).catch(e=>{console.error('Falha ao persistir',key,e);ui?.alert?.('O navegador não conseguiu salvar os dados. Exporte um backup antes de fechar a página.','Falha ao salvar')});return storageWriteChain}
+let autoBackups=[],cloudPendingQueue=[],storageWriteChain=Promise.resolve(),cloudQueueWriteChain=Promise.resolve(),cloudEnqueueChain=Promise.resolve(),cloudEnqueuePending=0,syncBaseCache=new Map();
+function enqueueStorageWrite(key,value){const snapshot=deepClone(value);storageWriteChain=storageWriteChain.then(()=>storage.set(key,snapshot)).catch(e=>{console.error('Falha ao persistir',key,e);Promise.resolve().then(()=>ui.alert('O navegador não conseguiu salvar os dados. Exporte um backup antes de fechar a página.','Falha ao salvar')).catch(err=>console.warn('Aviso de falha ao salvar',err))});return storageWriteChain}
 async function migrateLegacyStorage(){
   let main=await storage.get(APP_DB_KEY);if(!main){try{const raw=localStorage.getItem(APP_DB_KEY)||localStorage.getItem(OLD_DB_KEY);if(raw){main=JSON.parse(raw);await storage.set(APP_DB_KEY,main)}}catch(e){console.warn('Migração do banco legado',e)}}
@@ -479,9 +479,9 @@
 async function gzipText(text){
   if(!('CompressionStream' in window)||!('DecompressionStream' in window))return {raw:text};
-  try{const stream=new Blob([text]).stream().pipeThrough(new CompressionStream('gzip'));return {gzip:await new Response(stream).arrayBuffer()}}catch(e){console.warn('Compressão indisponível; salvando fila sem compressão.',e);return {raw:text}}
+  try{const stream=new Blob([text]).stream().pipeThrough(new CompressionStream('gzip'));return {gzip:Array.from(new Uint8Array(await new Response(stream).arrayBuffer()))}}catch(e){console.warn('Compressão indisponível; salvando fila sem compressão.',e);return {raw:text}}
 }
 async function gunzipText(payload){
   if(payload?.raw!==undefined)return String(payload.raw);
-  if(payload?.gzip&&'DecompressionStream' in window){const stream=new Blob([payload.gzip]).stream().pipeThrough(new DecompressionStream('gzip'));return await new Response(stream).text()}
+  if(payload?.gzip&&'DecompressionStream' in window){const bytes=Array.isArray(payload.gzip)?Uint8Array.from(payload.gzip):payload.gzip;if(!(bytes instanceof ArrayBuffer)&&!ArrayBuffer.isView(bytes))throw Error('Payload gzip inválido na fila offline.');const stream=new Blob([bytes]).stream().pipeThrough(new DecompressionStream('gzip'));return await new Response(stream).text()}
   throw Error('Fila compactada não pode ser lida neste navegador.');
 }
@@ -492,6 +492,6 @@
   try{const parsed=JSON.parse(await gunzipText(item.payload));return {id:item.id||uid(),workspaceId:item.workspaceId,createdAt:item.createdAt||new Date().toISOString(),data:parsed.data,baseData:parsed.baseData??null}}catch(e){console.warn('Item inválido na fila offline foi ignorado.',e);return null}
 }
-async function persistCloudQueue(){const packed=[];for(const item of cloudPendingQueue.slice(-10))packed.push(await packCloudQueueItem(item));await storage.set(CLOUD_QUEUE_KEY,packed)}
-function saveCloudQueue(){cloudQueueWriteChain=cloudQueueWriteChain.then(persistCloudQueue).catch(e=>console.error('Falha ao salvar fila offline',e));return cloudQueueWriteChain}
+async function persistCloudQueue(items){const packed=[];for(const item of items)packed.push(await packCloudQueueItem(item));await storage.set(CLOUD_QUEUE_KEY,packed)}
+function saveCloudQueue(){const snapshot=deepClone(cloudPendingQueue.slice(-10)),write=cloudQueueWriteChain.then(()=>persistCloudQueue(snapshot));cloudQueueWriteChain=write.catch(e=>console.error('Falha ao salvar fila offline',e));return write}
 
 async function loadPersistentState(){const legacyMain=await migrateLegacyStorage(),stored=await storage.get(APP_DB_KEY),raw=stored||legacyMain;let data;try{data=migrateData(raw||null)}catch(e){console.error(e);data=emptyDB()}await storage.set(APP_DB_KEY,data);const idx=await storage.get(AUTO_INDEX_KEY);autoBackups=[];if(Array.isArray(idx))for(const id of idx.slice(0,5)){const b=await storage.get(AUTO_ITEM_PREFIX+id);if(b)autoBackups.push(b)}const storedQueue=await storage.get(CLOUD_QUEUE_KEY)||[];cloudPendingQueue=[];if(Array.isArray(storedQueue))for(const item of storedQueue.slice(-10)){const unpacked=await unpackCloudQueueItem(item);if(unpacked)cloudPendingQueue.push(unpacked)}return data}
@@ -532,5 +532,13 @@
 function getAutoBackups(){return autoBackups.slice()}
 function saveAutoBackup(label){try{const snapshot={id:uid(),at:new Date().toISOString(),label:String(label||'Alteração'),data:deepClone(db)},removed=autoBackups.slice(4);autoBackups.unshift(snapshot);autoBackups=autoBackups.slice(0,5);enqueueStorageWrite(AUTO_ITEM_PREFIX+snapshot.id,snapshot);enqueueStorageWrite(AUTO_INDEX_KEY,autoBackups.map(b=>b.id));for(const b of removed)storage.remove(AUTO_ITEM_PREFIX+b.id).catch(()=>{})}catch(e){console.warn('backup automático',e)}}
-function persist(){db.version=SCHEMA_VERSION;db.updatedAt=new Date().toISOString();enqueueStorageWrite(APP_DB_KEY,db);renderCurrentPageOnly();updateSaveStatus();if(cloud.workspace){cloud.dirty=true;enqueueCloudSnapshot(normalizedData(db)).then(()=>{if(!cloud.busy)return processCloudQueue({silent:true})}).catch(e=>console.warn('Fila da nuvem',e))}}
+function persist(){
+  db.version=SCHEMA_VERSION;db.updatedAt=new Date().toISOString();
+  enqueueStorageWrite(APP_DB_KEY,db);renderCurrentPageOnly();updateSaveStatus();
+  if(cloud.workspace){
+    cloud.dirty=true;const snapshot=deepClone(db),workspaceId=cloud.workspace.id;
+    const enqueued=Promise.resolve().then(()=>enqueueCloudSnapshot(normalizedData(snapshot),workspaceId)).catch(e=>{console.warn('Fila de nuvem não enfileirada',e);return false});
+    Promise.resolve().then(()=>enqueued).then(ok=>{if(ok!==false&&!cloud.busy)return processCloudQueue({silent:true})}).catch(e=>console.warn('Falha ao processar fila de nuvem',e));
+  }
+}
 function commit(label,mutator){saveAutoBackup(label);undoStack.push({label,data:deepClone(db)});if(undoStack.length>20)undoStack.shift();mutator();db=migrateData(db);db.activityLog.push({id:uid(),ts:new Date().toISOString(),label,page:currentPage,actor:db.operatorName||'Você',createdAt:new Date().toISOString()});if(db.activityLog.length>300)db.activityLog=db.activityLog.slice(-300);persist();toast(label);updateUndo()}
 function undoLast(){let x=undoStack.pop();if(!x)return;saveAutoBackup('Antes de desfazer');db=migrateData(x.data);persist();toast('Alteração desfeita: '+x.label);updateUndo()}
@@ -823,5 +831,5 @@
 function updateConnectivity(){let el=$('connectivityStatus');if(!el)return;let on=navigator.onLine!==false;el.textContent=on?'Online':'Sem internet';el.classList.toggle('offline',!on)}
 function installApp(){if(deferredInstallPrompt){deferredInstallPrompt.prompt();deferredInstallPrompt.userChoice.finally(()=>{deferredInstallPrompt=null})}else ui.alert('Se o botão de instalação do navegador não aparecer:\n\nAndroid/Chrome: menu ⋮ → Instalar app / Adicionar à tela inicial.\n\niPhone/Safari: Compartilhar → Adicionar à Tela de Início.')}
-window.addEventListener('beforeinstallprompt',e=>{e.preventDefault();deferredInstallPrompt=e;toast('ReceitaPro pode ser instalado como aplicativo.')});window.addEventListener('online',()=>{updateConnectivity();processCloudQueue({silent:true})});window.addEventListener('offline',updateConnectivity);
+window.addEventListener('beforeinstallprompt',e=>{e.preventDefault();deferredInstallPrompt=e;toast('ReceitaPro pode ser instalado como aplicativo.')});window.addEventListener('online',()=>{updateConnectivity();Promise.resolve().then(()=>processCloudQueue({silent:true})).catch(e=>console.warn('Falha ao retomar fila de nuvem',e))});window.addEventListener('offline',updateConnectivity);
 async function initPWA(){updateConnectivity();if(!('serviceWorker' in navigator)||!location.protocol.startsWith('http'))return;try{const reg=await navigator.serviceWorker.register('./sw.js');const announce=()=>toast('Nova versão disponível. Recarregue.');if(reg.waiting&&navigator.serviceWorker.controller)announce();reg.addEventListener('updatefound',()=>{const worker=reg.installing;if(!worker)return;worker.addEventListener('statechange',()=>{if(worker.state==='installed'&&navigator.serviceWorker.controller)announce()})})}catch(e){console.warn('Service worker não pôde ser registrado.',e)}}
 
@@ -868,15 +876,77 @@
 async function cloudCreateWorkspace(){let name=$('workspaceName').value.trim();if(!name)return ui.alert('Digite o nome do negócio.');try{let w=await cloudRequest('/rest/v1/rpc/gp_create_workspace','POST',{p_name:name});await cloudOpenWorkspace(w);cloudView()}catch(e){ui.alert(e.message)}}
 async function cloudJoinWorkspace(){let code=$('workspaceCode').value.trim().toUpperCase();if(!code)return ui.alert('Digite o código de convite.');try{let w=await cloudRequest('/rest/v1/rpc/gp_join_workspace','POST',{p_invite_code:code});await cloudOpenWorkspace(w);cloudView()}catch(e){ui.alert(e.message)}}
-async function cloudOpenWorkspace(w){const base=await loadSyncBase(w.id),local=deepClone(normalizedData(db)),localHas=hasBusinessData(local),state=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:w.id}),remote=normalizedData(state?.data),remoteHas=hasBusinessData(remote);cloud.workspace=w;cloud.version=state.version;cloud.dirty=false;if(sameValue(local,remote)){db=remote;localPersist();saveSyncBase()}else if(localHas&&!remoteHas){if(!await ui.confirm('Enviar dados para a nuvem','A nuvem está vazia e este aparelho tem dados. Enviar os dados deste aparelho para a nuvem?')){cloud.workspace=null;cloudView();return false}db=local;localPersist();cloud.dirty=true;if(!await cloudSave())return false}else if(!localHas&&remoteHas){db=remote;localPersist();saveSyncBase()}else if(localHas&&remoteHas){if(!await ui.confirm('Juntar dados locais e da nuvem','Há dados neste aparelho e na nuvem. Deseja juntar os dois com proteção contra ingredientes duplicados?')){cloud.workspace=null;cloudView();return false}let merged=mergeBusinessData(remote,local,base?.data||null);db=merged.data;localPersist();cloud.dirty=true;if(!await cloudSave())return false;toast(merged.conflicts?`Dados unidos. ${merged.conflicts} conflito(s) resolvido(s) com a versão deste aparelho.`:'Dados unidos e sincronizados.')}else{db=remote;localPersist();saveSyncBase()}if(cloud.poll)clearInterval(cloud.poll);cloud.poll=setInterval(()=>{if(!cloud.busy)cloudRefresh(true)},15000);cloudView();return true}
+async function cloudOpenWorkspace(w){const base=await loadSyncBase(w.id),local=deepClone(normalizedData(db)),localHas=hasBusinessData(local),state=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:w.id}),remote=normalizedData(state?.data),remoteHas=hasBusinessData(remote);cloud.workspace=w;cloud.version=state.version;cloud.dirty=false;if(sameValue(local,remote)){db=remote;localPersist();saveSyncBase()}else if(localHas&&!remoteHas){if(!await ui.confirm('Enviar dados para a nuvem','A nuvem está vazia e este aparelho tem dados. Enviar os dados deste aparelho para a nuvem?')){cloud.workspace=null;cloudView();return false}db=local;localPersist();cloud.dirty=true;if(!await cloudSave())return false}else if(!localHas&&remoteHas){db=remote;localPersist();saveSyncBase()}else if(localHas&&remoteHas){if(!await ui.confirm('Juntar dados locais e da nuvem','Há dados neste aparelho e na nuvem. Deseja juntar os dois com proteção contra ingredientes duplicados?')){cloud.workspace=null;cloudView();return false}let merged=mergeBusinessData(remote,local,base?.data||null);db=merged.data;localPersist();cloud.dirty=true;if(!await cloudSave())return false;toast(merged.conflicts?`Dados unidos. ${merged.conflicts} conflito(s) resolvido(s) com a versão deste aparelho.`:'Dados unidos e sincronizados.')}else{db=remote;localPersist();saveSyncBase()}if(cloud.poll)clearInterval(cloud.poll);cloud.poll=setInterval(()=>{if(!cloud.busy)Promise.resolve().then(()=>cloudRefresh(true)).catch(e=>console.warn('Falha ao atualizar nuvem',e))},15000);cloudView();return true}
 function localPersist(){enqueueStorageWrite(APP_DB_KEY,db);renderCurrentPageOnly()}
-async function cloudRefresh(silent=false){if(!cloud.workspace||cloud.busy)return;try{let state=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:cloud.workspace.id}),remote=normalizedData(state.data);if(cloud.dirty){let base=await loadSyncBase(cloud.workspace.id),merged=mergeBusinessData(remote,db,base?.data||null);cloud.version=state.version;db=merged.data;localPersist();await cloudSave({silent:true})}else if(state.version!==cloud.version){cloud.version=state.version;db=remote;localPersist();saveSyncBase();if(!silent)toast('Dados atualizados da nuvem.')}}catch(e){if(!silent)toast('Sem conexão com a nuvem. Seus dados continuam salvos neste aparelho.')}}
-async function enqueueCloudSnapshot(snapshot=normalizedData(db)){if(!cloud.workspace?.id)return;const wid=cloud.workspace.id,last=cloudPendingQueue[cloudPendingQueue.length-1];if(last&&last.workspaceId===wid&&sameValue(last.data,snapshot)){cloud.pending=true;return}const base=(await loadSyncBase(wid))?.data||null;cloudPendingQueue.push({id:uid(),workspaceId:wid,createdAt:new Date().toISOString(),data:deepClone(snapshot),baseData:base?deepClone(base):null});if(cloudPendingQueue.length>10)cloudPendingQueue=cloudPendingQueue.slice(-10);cloud.pending=true;saveCloudQueue()}
-async function saveSnapshotToCloud(item,options={}){let lastError=null;for(let attempt=0;attempt<3;attempt++){try{const latest=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:cloud.workspace.id}),remote=normalizedData(latest?.data),base=item.baseData?normalizedData(item.baseData):((await loadSyncBase(cloud.workspace.id))?.data||null),merged=sameValue(remote,item.data)?{data:normalizedData(item.data),conflicts:0}:mergeBusinessData(remote,item.data,base),payload=merged.data;cloud.version=latest.version;const result=await cloudRequest('/rest/v1/rpc/gp_save_data','POST',{p_workspace_id:cloud.workspace.id,p_expected_version:cloud.version,p_data:payload});cloud.version=result.version;saveSyncBase(payload,cloud.version);return {ok:true,data:payload,conflicts:merged.conflicts}}catch(e){lastError=e;if(!/CONFLICT_VERSION/i.test(String(e.message||'')))break}}throw lastError||Error('Falha ao sincronizar')}
-async function processCloudQueue(options={}){if(!cloud.workspace||cloud.busy||navigator.onLine===false)return false;cloud.busy=true;document.body.classList.add('busy');let success=true,lastSynced=null;try{while(true){const idx=cloudPendingQueue.findIndex(x=>x.workspaceId===cloud.workspace.id);if(idx<0)break;const item=cloudPendingQueue[idx];try{const result=await saveSnapshotToCloud(item,options);lastSynced=result.data;cloudPendingQueue.splice(idx,1);saveCloudQueue()}catch(e){success=false;cloud.dirty=true;if(!options.silent)toast('Nuvem indisponível. A alteração ficou na fila offline.');break}}if(lastSynced&&!cloudPendingQueue.some(x=>x.workspaceId===cloud.workspace.id)){db=normalizedData(lastSynced);localPersist()}cloud.dirty=cloudPendingQueue.some(x=>x.workspaceId===cloud.workspace.id);cloud.pending=cloud.dirty;cloudView();return success&&!cloud.dirty}finally{cloud.busy=false;document.body.classList.remove('busy');const rerun=cloud.pending&&navigator.onLine!==false;if(rerun)queueMicrotask(()=>processCloudQueue({silent:true}))}}
-async function cloudSave(options={}){if(!cloud.workspace)return false;await enqueueCloudSnapshot(normalizedData(db));return processCloudQueue(options)}
+async function cloudRefresh(silent=false){if(!cloud.workspace||cloud.busy)return;try{let state=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:cloud.workspace.id}),remote=normalizedData(state.data);if(cloud.dirty){let base=await loadSyncBase(cloud.workspace.id),merged=mergeBusinessData(remote,db,base?.data||null);cloud.version=state.version;db=merged.data;localPersist();await cloudSave({silent:true})}else if(state.version!==cloud.version){cloud.version=state.version;db=remote;localPersist();saveSyncBase();if(!silent)toast('Dados atualizados da nuvem.')}}catch(e){console.warn('Falha ao atualizar nuvem',e);if(!silent)toast('Sem conexão com a nuvem. Seus dados continuam salvos neste aparelho.')}}
+async function enqueueCloudSnapshot(snapshot=normalizedData(db),workspaceId=cloud.workspace?.id){
+  if(!workspaceId)return false;
+  const data=deepClone(snapshot);cloudEnqueuePending++;
+  const run=cloudEnqueueChain.then(async()=>{
+    const base=(await loadSyncBase(workspaceId))?.data||null,last=cloudPendingQueue[cloudPendingQueue.length-1];
+    if(!last||last.workspaceId!==workspaceId||!sameValue(last.data,data)){
+      cloudPendingQueue.push({id:uid(),workspaceId,createdAt:new Date().toISOString(),data,baseData:base?deepClone(base):null});
+      if(cloudPendingQueue.length>10)cloudPendingQueue=cloudPendingQueue.slice(-10);
+    }
+    if(cloud.workspace?.id===workspaceId){cloud.pending=true;cloud.dirty=true}
+    await saveCloudQueue();return true;
+  });
+  cloudEnqueueChain=run.catch(e=>console.warn('Falha no enfileiramento de nuvem',e));
+  try{return await run}finally{cloudEnqueuePending--}
+}
+async function saveSnapshotToCloud(item,options={}){
+  const workspaceId=item.workspaceId;let lastError=null;
+  for(let attempt=0;attempt<3;attempt++){
+    try{
+      if(cloud.workspace?.id!==workspaceId)throw Error('Empresa mudou durante a sincronização.');
+      const latest=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:workspaceId}),remote=normalizedData(latest?.data),base=item.baseData?normalizedData(item.baseData):((await loadSyncBase(workspaceId))?.data||null),merged=sameValue(remote,item.data)?{data:normalizedData(item.data),conflicts:0}:mergeBusinessData(remote,item.data,base),payload=merged.data;
+      if(cloud.workspace?.id!==workspaceId)throw Error('Empresa mudou durante a sincronização.');
+      const result=await cloudRequest('/rest/v1/rpc/gp_save_data','POST',{p_workspace_id:workspaceId,p_expected_version:latest.version,p_data:payload});
+      if(cloud.workspace?.id!==workspaceId)throw Error('Empresa mudou durante a sincronização.');
+      cloud.version=result.version;saveSyncBase(payload,cloud.version);return {ok:true,data:payload,conflicts:merged.conflicts};
+    }catch(e){lastError=e;if(!/CONFLICT_VERSION/i.test(String(e.message||'')))break}
+  }
+  throw lastError||Error('Falha ao sincronizar');
+}
+async function processCloudQueue(options={}){
+  const workspaceId=cloud.workspace?.id;
+  if(!workspaceId||cloud.busy||navigator.onLine===false)return false;
+  cloud.busy=true;document.body.classList.add('busy');let success=true,lastSynced=null,lastSource=null;
+  try{
+    await cloudEnqueueChain;await saveCloudQueue();
+    while(cloud.workspace?.id===workspaceId&&navigator.onLine!==false){
+      await cloudEnqueueChain;
+      const item=cloudPendingQueue.find(x=>x.workspaceId===workspaceId);if(!item)break;
+      const result=await saveSnapshotToCloud(item,options);
+      if(cloud.workspace?.id!==workspaceId){success=false;break}
+      lastSynced=result.data;lastSource=item.data;
+      const idx=cloudPendingQueue.findIndex(x=>x.id===item.id);
+      if(idx>=0){
+        cloudPendingQueue.splice(idx,1);
+        try{await saveCloudQueue()}catch(e){if(!cloudPendingQueue.some(x=>x.id===item.id))cloudPendingQueue.splice(Math.min(idx,cloudPendingQueue.length),0,item);await saveCloudQueue().catch(err=>console.warn('Falha ao restaurar fila offline',err));throw e}
+      }
+    }
+    await cloudEnqueueChain;
+    if(cloud.workspace?.id===workspaceId){
+      const queued=cloudPendingQueue.some(x=>x.workspaceId===workspaceId);
+      // Uma resposta antiga nunca substitui alterações feitas durante a sincronização.
+      if(lastSynced&&!queued&&!cloudEnqueuePending&&sameValue(normalizedData(db),normalizedData(lastSource))){db=normalizedData(lastSynced);localPersist()}
+      cloud.dirty=queued||cloudEnqueuePending>0;cloud.pending=cloud.dirty;cloudView();
+    }else success=false;
+    return success&&!cloud.dirty;
+  }catch(e){
+    success=false;console.warn('Falha ao processar fila de nuvem',e);
+    if(cloud.workspace?.id===workspaceId){cloud.dirty=true;cloud.pending=true;cloudView()}
+    if(!options.silent)toast('Nuvem indisponível. A alteração ficou na fila offline.');return false;
+  }finally{
+    cloud.busy=false;document.body.classList.remove('busy');
+    const changedWorkspace=cloud.workspace?.id&&cloud.workspace.id!==workspaceId;
+    if((success||changedWorkspace)&&cloud.workspace&&navigator.onLine!==false&&(cloud.pending||cloudEnqueuePending))Promise.resolve().then(()=>processCloudQueue({silent:true})).catch(e=>console.warn('Falha ao continuar fila de nuvem',e));
+  }
+}
+async function cloudSave(options={}){if(!cloud.workspace)return false;const enqueued=await enqueueCloudSnapshot(normalizedData(db)).catch(e=>{console.warn('Fila de nuvem não enfileirada',e);return false});if(enqueued===false)return false;return processCloudQueue(options)}
 function cloudLogout(){if(cloud.poll)clearInterval(cloud.poll);cloud.poll=null;cloud.token='';cloud.refresh='';cloud.user=null;cloud.workspace=null;cloud.dirty=false;sessionStorage.removeItem('receitaProAuth');cloudView();toast('Você saiu da conta. Os dados locais continuam disponíveis.')}
 function cloudDisconnect(){cloudLogout();cloud.url='';cloud.key='';localStorage.removeItem('receitaProCloudConfig');cloudView()}
 function copyInvite(){navigator.clipboard?.writeText(cloud.workspace?.invite_code||'').then(()=>toast('Código copiado.')).catch(()=>toast('Selecione e copie o código manualmente.'))}
-function initCloud(){try{let cfg=JSON.parse(localStorage.getItem('receitaProCloudConfig')||localStorage.getItem('geladinhoCloudConfig')||'null');if(cfg?.url&&cfg?.key){cloud.url=cfg.url;cloud.key=cfg.key;$('cloudUrl').value=cfg.url;$('cloudKey').value=cfg.key}let auth=JSON.parse(sessionStorage.getItem('receitaProAuth')||sessionStorage.getItem('geladinhoAuth')||'null');if(auth?.token&&cloud.url){cloud.token=auth.token;cloud.refresh=auth.refresh;cloud.user=auth.user;sessionStorage.setItem('receitaProAuth',JSON.stringify(auth));cloudLoadMyWorkspaces().catch(()=>cloudLogout())}}catch(e){console.warn(e)}cloudView()}
+function initCloud(){try{let cfg=JSON.parse(localStorage.getItem('receitaProCloudConfig')||localStorage.getItem('geladinhoCloudConfig')||'null');if(cfg?.url&&cfg?.key){cloud.url=cfg.url;cloud.key=cfg.key;$('cloudUrl').value=cfg.url;$('cloudKey').value=cfg.key}let auth=JSON.parse(sessionStorage.getItem('receitaProAuth')||sessionStorage.getItem('geladinhoAuth')||'null');if(auth?.token&&cloud.url){cloud.token=auth.token;cloud.refresh=auth.refresh;cloud.user=auth.user;sessionStorage.setItem('receitaProAuth',JSON.stringify(auth));cloudLoadMyWorkspaces().catch(e=>{console.warn('Falha ao abrir empresas da nuvem',e);cloudLogout()})}}catch(e){console.warn(e)}cloudView()}
 
 
@@ -892,8 +962,8 @@
   $('posSearch')?.addEventListener('input',renderPOS);
   document.addEventListener('keydown',e=>{const tag=document.activeElement?.tagName,editing=['INPUT','TEXTAREA','SELECT'].includes(tag);let k=e.key.toLowerCase();if(k==='?'&&!editing){e.preventDefault();openShortcutHelp();return}if(!(e.altKey||e.ctrlKey))return;if(e.ctrlKey&&k==='k'){e.preventDefault();openGlobalSearch()}else if(e.altKey&&k==='v'){e.preventDefault();goPage('sales');focusId('saleQty')}else if(e.altKey&&k==='p'){e.preventDefault();goPage('production');setProductionTab('register');focusId('prodQty')}else if(e.altKey&&k==='l'){e.preventDefault();goPage('production');setProductionTab('planner');focusId('planQty')}else if(e.ctrlKey&&k==='z'&&!e.shiftKey&&!editing){e.preventDefault();undoLast()}});
-  if(shouldShowOnboarding()&&!launchAction)setTimeout(openOnboarding,250);setTimeout(checkLocalReminders,1200);if(navigator.onLine!==false&&cloudPendingQueue.length)setTimeout(()=>processCloudQueue({silent:true}),800);
+  if(shouldShowOnboarding()&&!launchAction)setTimeout(openOnboarding,250);setTimeout(checkLocalReminders,1200);if(navigator.onLine!==false&&cloudPendingQueue.length)setTimeout(()=>{Promise.resolve().then(()=>processCloudQueue({silent:true})).catch(e=>console.warn('Falha ao iniciar fila de nuvem',e))},800);
   console.info('ReceitaPro 6.1 — schema v11, PWA real, fila offline compactada, acessibilidade e renders por página');
 }
 const exposed={addRecipeItem,clearAll,clearDayPlan,cloudConfigure,cloudCreateWorkspace,cloudDisconnect,cloudJoinWorkspace,cloudLogin,cloudLogout,cloudRefresh,copyDayShoppingList,copyInvite,copyShoppingList,exportData,exportInventoryCSV,exportSalesCSV,fillMaxProduction,fillOnePackage,fillPackageCount,focusId,goPage,installApp,openGlobalSearch,openLossDialog,openQuickAction,openWalletSetup,planToProduction,quickGo,renderHealth,resetExpenseForm,resetIngredientForm,resetProductionForm,resetPurchaseForm,resetRecipeForm,resetSaleForm,setInventoryTab,setProductionTab,toggleInitialStock,toggleTheme,toggleUIMode,undoLast,useMaxPlan,openShortcutHelp,enableNotifications,openOnboarding,openPOS,closePOS,addPOSItem,changePOSQty,clearPOSCart,finishPOS,printDailyReport,addDayPlanLot,deleteAdjustment,deleteClosing,deleteExpense,deleteIngredient,deleteProductAdjustment,deleteProduction,deletePurchase,deleteRecipe,deleteSale,deleteWalletMovement,duplicateRecipe,editExpense,editIngredient,editProduction,editPurchase,editRecipe,editSale,openReceiveDialog,openWalletMove,planRecipe,prepareQuickSale,quickProduce,restoreAutoBackup,setDayPlanQty,startCount,toggleFavoriteRecipe,useSuggestedPrice};Object.assign(window,exposed);
 init().catch(err=>{console.error(err);ui.alert('O ReceitaPro não conseguiu iniciar corretamente. Recarregue a página e, se o problema continuar, restaure um backup.','Erro ao iniciar').catch(()=>{})});
-</script></body></html>+</script></body></html>
```


## Etapa 2 — Bloco A

A1–A4: aplicado/verificado. O anexo já continha estes comportamentos; foram conservados. Funções: `init`, `persist`, `loadSyncBase`, `renderHealth`, `initPWA`.

### A1 — eventos dos botões

ANTES → DEPOIS (mesmo trecho real, verificado):

```js
$('sideCloudBtn')?.addEventListener('click',()=>{goPage('settings');setTimeout(()=>$('cloudSection')?.scrollIntoView({behavior:'smooth'}),100)});
$('dailyCloseBtn')?.addEventListener('click',()=>{goPage('finance');setTimeout(()=>$('closingForm')?.scrollIntoView({behavior:'smooth'}),100)});
```

Nenhum onclick com seta codificada ficou nesses botões. Manual: clicar em Nuvem e backup e Fechar o dia; conferir página e seção no viewport.

### A2 — contrato assíncrono

ANTES → DEPOIS: `async function loadSyncBase(id)` continua assíncrona; `function persist()` e `function commit(label,mutator)` continuam síncronas. Os corpos reais e o diff estão na Etapa 1.

A nuvem aguarda sua cadeia internamente, sem tornar operações locais promises. Manual: cadastrar offline/recarregar; simular atraso da base e verificar retorno imediato do commit.

### A3 — diagnóstico sob demanda

ANTES → DEPOIS (trecho real preservado no init):

```js
if($('healthList'))$('healthList').innerHTML='<div class="health-row"><div><b>Verificação sob demanda</b><span>Clique em “Verificar agora” para rodar a análise do sistema.</span></div><span class="pill good">Pronto</span></div>';
```

Não há chamada de renderHealth no init. A verificação permanece no botão e nos fluxos de restauração/limpeza. Manual: abrir o app e observar mensagem sob demanda; clicar em Verificar agora.

### A4 — PWA sem bloquear inicialização

ANTES → DEPOIS (mesmo trecho real):

```js
initPWA().catch(e=>console.warn('PWA',e));
```

Não foi introduzido await de PWA no init. Manual: usar o HTML sem recursos PWA publicados e verificar que a UI/local continuam operando; instalação PWA completa é uma validação separada, pendente dos recursos ausentes.


## Etapa 3 — Bloco B

Arquivo alterado: `ReceitaPro_6.2_FINAL.html`. Somente itens B1–B6; sem mudanças em Bloco C/UX nesta edição.

### B1 — aparência consolidada da sidebar

**Status:** aplicado. CSS global `.sidebar`, `.nav button`, tokens `:root`/dark e ajustes de tipografia secundária. A sidebar tem uma única declaração global; cores variam por tokens, sem bloco min-width:721px de sobrescrita. O menu móvel mantém sua posição inferior e usa as mesmas cores com contraste AA; tamanhos/layout ficam nos media queries.

```diff
--- ANTES
+++ DEPOIS
@@ -1,2 +1,2 @@
-.sidebar{position:sticky;top:0;height:100vh;background:#0f3b2f;color:#d9eee5;padding:22px 14px;display:flex;flex-direction:column;z-index:30}
-.nav button.active{background:#e6f4eb;color:#164d3d}
+.sidebar{position:sticky;top:0;width:var(--sidebar);height:100vh;background:var(--sidebar-bg);color:var(--sidebar-text);padding:22px 14px;display:flex;flex-direction:column;z-index:30}
+.nav button.active{background:var(--sidebar-active-bg);color:#ffffff;box-shadow:inset 3px 0 0 var(--sidebar-active-marker)}
```
Tokens finais reais, nos temas correspondentes:

```css
--sidebar:236px;--sidebar-bg:#0b3a2d;--sidebar-text:#eef8f4;--sidebar-nav-text:#d9ebe4;--sidebar-active-bg:#165b46;--sidebar-active-marker:#7bc6a7
--sidebar-bg:#071f18;--sidebar-text:#eef8f4;--sidebar-nav-text:#cfe4da;--sidebar-active-bg:#124332;--sidebar-active-marker:#62c49d
```

Contrastes calculados (mínimo AA texto comum: 4,5:1): claro texto 11.69:1, claro nav 10.23:1, claro ativo 8.02:1, dark texto 15.9:1, dark nav 12.94:1, dark ativo 11.2:1.

Teste manual: alternar claro/escuro em desktop e telefone, verificar texto/nav/estado ativo e contraste sem inversão clara acidental.

### B2 — largura por uma variável

**Status:** aplicado. `--sidebar:236px` permanece no root; o breakpoint redefine 210px e `.app`/sidebar usam `var(--sidebar)`. O override do root está depois da definição padrão, evitando que a cascade restaure 236px em telas médias; o override móvel 100% permanece.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1,2 @@
-.sidebar{width:210px}.app{grid-template-columns:210px minmax(0,1fr)}
+.app>.sidebar {width:var(--sidebar)}.app{grid-template-columns:var(--sidebar) minmax(0,1fr)}
+@media(max-width:1100px){:root{--sidebar:210px}}
```
Teste manual: largura 1280px → sidebar236px; 900px → sidebar210px; 390px → header largura100%, menu inferior sem corte e sete itens navegáveis.

### B3 — dialog redundante

**Status:** aplicado. Removido somente o markup `alertDialog`; auditoria literal confirmou ausência de referências no JavaScript. `ui.alert`, fila de diálogos e fallbacks permanecem.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +0,0 @@
-<dialog aria-labelledby="alertDialogTitle" aria-modal="true" id="alertDialog" role="dialog"><div class="dialog-body"><h3 id="alertDialogTitle">Aviso</h3><p id="alertDialogText"></p></div><div class="dialog-actions"><button class="btn primary" id="alertDialogOk" type="button">OK</button></div></dialog>
```
Teste manual: invocar uma validação que usa ui.alert e confirmar que o diálogo aparece e fecha; verificar que não existe ID alertDialog no DOM.

### B4 — gravar fila antes de prosseguir

**Status:** aplicado na Etapa 1 e preservado. `enqueueCloudSnapshot` continua async com `await saveCloudQueue()`; `persist()` continua síncrona e agenda nuvem por `Promise.resolve().then(...).catch(...)`. Não houve nova modificação neste item.

Trechos reais em vigor:

```js
const enqueued=Promise.resolve().then(()=>enqueueCloudSnapshot(normalizedData(snapshot),workspaceId)).catch(e=>{console.warn('Fila de nuvem não enfileirada',e);return false});
await saveCloudQueue();return true;
```

Teste manual: atrasar storage da fila e confirmar que envio da nuvem só começa depois da gravação; cadastro local deve responder e persistir sem aguardar a nuvem.

### B5 — Maps com chaves únicas

**Status:** aplicado. `currentDashboardActions`/`currentSearchActions`, renderizadores e listeners delegados. Cada render limpa seu Map e gera chaves monotônicas string; um nó antigo não consegue executar a ação de um resultado novo. Os handlers consultam `.get(key)` e usam os novos atributos `data-*-action-key`.

```diff
--- ANTES
+++ DEPOIS
@@ -1,5 +1,5 @@
-currentSearchActions=[],currentDashboardActions=[];
-const idx=a.run?currentDashboardActions.push(a.run)-1:-1;
-const fn=currentDashboardActions[Number(b.dataset.dashboardActionIndex)];
-const idx=currentSearchActions.push(r.run)-1;
-const fn=currentSearchActions[Number(b.dataset.searchActionIndex)];
+currentSearchActions=new Map(),currentDashboardActions=new Map(),uiActionKeySequence=0,revenueChartRenderCount=0;
+const key=a.run?`dashboard-${++uiActionKeySequence}`:'';if(a.run)currentDashboardActions.set(key,a.run);
+const fn=currentDashboardActions.get(b.dataset.dashboardActionKey);
+const key=`search-${++uiActionKeySequence}`;currentSearchActions.set(key,r.run);
+const fn=currentSearchActions.get(b.dataset.searchActionKey);
```
O browser gate também reproduziu um defeito anterior ao Map: `ReferenceError: closeSearchDialog is not defined` no callback de busca (`renderGlobalSearch`, original linha755:549; listener757:209). Adicionado o helper interno abaixo para restaurar fechamento/navegação em todos os resultados; nenhum call site inline exige expor a função.

```diff
+function closeSearchDialog(){const d=$('searchDialog');if(d?.open)d.close()}
```

Teste manual: clicar alerta Começar, buscar e clicar um resultado; repetir os renders e verificar Maps/chaves novas, sem ação associada a key antiga. Nenhuma função exposta foi renomeada ou removida.

### B6 — gradient único por render

**Status:** aplicado. `drawRevenueChart()` usa contador de render para gerar o ID e referencia o mesmo valor no polígono SVG. ARIA, dados e aparência do gráfico permanecem.

```diff
--- ANTES
+++ DEPOIS
@@ -1,2 +1,3 @@
-<linearGradient id="fillChart" x1="0" y1="0" x2="0" y2="1">
-fill="url(#fillChart)"
+const gradientId=`revenueChartGradient-${++revenueChartRenderCount}`;
+<linearGradient id="${gradientId}" x1="0" y1="0" x2="0" y2="1">
+fill="url(#${gradientId})"
```
Teste manual: chamar drawRevenueChart duas vezes, confirmar IDs diferentes e que fill referencia o ID atual; repetir alternando tema.

### Validação e checkpoint

- Node `--check`: aprovado.
- Estática B1–B6: aprovada; única `.sidebar{`, nenhum min721 que sobrepõe cores, sem largura210px hardcoded, sem alertDialog, nenhum array/index legado de ações, gradient gerado, `await saveCloudQueue` preservado.
- Browser checkpoint de Maps, largura/contraste e fluxo persistente: passou (12 verificações B e fluxo a–j repetido).
- `window.exposed`: nenhuma função movida, removida ou renomeada; Maps são internas ao módulo e listeners atualizados.
- Handlers de dashboard/busca preservados com novas chaves; nenhum handler removido sem equivalente.
- Bloco C e Etapa5 não implementados nesta edição.


## Etapa 4 — Bloco C

Arquivo alterado: `ReceitaPro_6.2_FINAL.html`. Todos os itens C1–C9 tratados; funcionalidades existentes preservadas. Nenhuma mudança de Etapa5/schema nesta edição.

### C1 — foco unificado

**Status:** aplicado. CSS: removida alteração de borda específica dos campos focados; mantido um único `*:focus-visible` com outline 2px, deslocado depois do reset de inputs para garantir precedência. Bordas normais/validação permanecem; teclado usa uma indicação de foco consistente.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +0,0 @@
-.input:focus-visible,select:focus-visible,textarea:focus-visible{border-color:var(--color-brand)}
```

Trecho final após o reset dos campos:

```css
*:focus-visible{outline:2px solid var(--color-brand);outline-offset:2px;border-radius:6px}
```

Teste manual: usar Tab para focar campo válido, confirmar outline2px e que border-color permanece igual ao estado sem foco; verificar botões/selects.

### C2 — teclado numérico correto

**Status:** aplicado. Os 28 campos pedidos recebem `inputmode="numeric"` para step1 ou `inputmode="decimal"` para step0.01/any; o campo dinâmico `.riQty` também recebe decimal. Tipos, limites e validações originais permanecem.

```diff
--- ANTES
+++ DEPOIS
@@ -1,3 +1,3 @@
-<input class="input" id="purchasePacks" min="0" placeholder="1" step="1" type="number" value="1"/>
-<input class="input" id="ingPackQty" min="0.001" placeholder="1000" required="" step="any" type="number"/>
-<input class="input" id="saleReceived" min="0" step="0.01" type="number"/>
+<input inputmode="numeric" class="input" id="purchasePacks" min="0" placeholder="1" step="1" type="number" value="1"/>
+<input inputmode="decimal" class="input" id="ingPackQty" min="0.001" placeholder="1000" required="" step="any" type="number"/>
+<input inputmode="decimal" class="input" id="saleReceived" min="0" step="0.01" type="number"/>
```

Teste manual: no celular, focar pacotes/produção para teclado inteiro e preço/peso/valor para teclado decimal; cadastrar normalmente valores fracionários.

### C3 — autocomplete

**Status:** aplicado. `saleCustomer` e `operatorName` usam name; `businessName` usa organization.

```diff
--- ANTES
+++ DEPOIS
@@ -1,3 +1,3 @@
-<input class="input" id="saleCustomer" placeholder="Ex.: João"/>
-<input class="input" id="operatorName" placeholder="Ex.: Vinícius"/>
-<input class="input" id="businessName" placeholder="Ex.: Sacolés do Vini"/>
+<input autocomplete="name" class="input" id="saleCustomer" placeholder="Ex.: João"/>
+<input autocomplete="name" class="input" id="operatorName" placeholder="Ex.: Vinícius"/>
+<input autocomplete="organization" class="input" id="businessName" placeholder="Ex.: Sacolés do Vini"/>
```

Teste manual: focar os três campos e conferir o preenchimento sugerido pelo navegador, quando existir perfil salvo.

### C4 — handlers nomeados e eventos

**Status:** aplicado. Atributos onclick complexos foram substituídos por `data-action`, funções nomeadas, `delegatedActions` e listener já existente. Casos: abrir carteira com scroll atrasado; navegação com foco; compra; abrir arquivo; fechar os dez botões de dialog; fechar quickDialog e abrir busca/perda. Helpers novos estão em `exposed`; chamadas inline simples de funções expostas continuam.

```diff
--- ANTES
+++ DEPOIS
@@ -1,3 +1,3 @@
-onclick="goPage('finance');focusId('expName')"
-onclick="document.getElementById('quickDialog').close();openLossDialog()"
-onclick="document.getElementById('searchDialog').close()"
+data-action="navigateWithFocus" data-id="finance" data-arg="expName"
+data-action="quickDialogAction" data-id="loss"
+data-action="closeDialog" data-id="searchDialog"
```

Helper dinâmico e equivalência real:

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1,2 @@
-onclick="goPage(\'finance\');setTimeout(()=>document.getElementById(\'walletFinanceCard\').scrollIntoView({behavior:\'smooth\'}),80)"
+data-action="openWalletFinance"
+function openWalletFinance(){goPage('finance');setTimeout(()=>$('walletFinanceCard')?.scrollIntoView({behavior:'smooth'}),80)}
```

Também substituídas propriedades `.onclick` em appDialog, receita, próximos passos e próximas ações por addEventListener/delegação. appDialog remove seus listeners ao finalizar; botões recriados recebem um listener cada, evitando duplicações nos renders. Não foi usado eval/new Function.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-div.querySelector('button').onclick=()=>{div.remove();previewRecipe()};
+div.querySelector('button').addEventListener('click',()=>{div.remove();previewRecipe()});
```

Teste manual: abrir/fechar dialogs; usar ação rápida Perda/Busca; Ver carteira deve navegar e fazer scroll; importar arquivo deve abrir seletor; próximo passo e próxima ação devem navegar uma única vez após vários renders; confirmar/cancelar alertas repetidamente.

### C5 — ARIA do wizard

**Status:** aplicado previamente e preservado. Quatro botões possuem aria-controls para o painel correspondente; quatro painéis têm ID, role=tabpanel e aria-labelledby do tab. Não houve mudança necessária nesta edição.

Trechos reais existentes:

```html
<button aria-controls="ingStepPanel1" aria-current="step" aria-selected="true" class="ingredient-step active" data-ing-step-target="1" id="ingStepTab1" role="tab" type="button">
<div aria-labelledby="ingStepTab1" class="guided-block" data-ing-step="1" id="ingStepPanel1" role="tabpanel">
```

Teste manual: inspecionar a árvore de acessibilidade enquanto troca as quatro etapas; ARIA deve relacionar botão e painel sem IDs repetidos.

### C6 — impressão por media query

**Status:** aplicado. Regras de impressão não dependem de body.printing; a seção printReport torna-se o único conteúdo visível em print. Estilos de relatório/tabela originais permanecem, assim como o botão imprimir.

```diff
--- ANTES
+++ DEPOIS
@@ -1,2 +1,2 @@
-body.printing>*:not(#printReport){display:none!important}
-body.printing #printReport{display:block!important;color:#111;background:#fff;padding:24px;font:14px/1.5 system-ui,sans-serif}
+body > *:not(#printReport){display:none!important}
+#printReport{display:block!important;color:#111;background:#fff;padding:24px;font:14px/1.5 system-ui,sans-serif}
```

Teste manual: gerar fechamento diário, abrir impressão e conferir somente relatório; usar preview print sem a classe printing e verificar a regra ativa.

### C7 — gráfico acessível

**Status:** aplicado previamente e preservado. drawRevenueChart define role=img e aria-label no container; SVG tem aria-hidden=true sem role. Nenhuma modificação adicional.

Trechos reais:

```js
el.setAttribute('role','img');
el.setAttribute('aria-label',`Gráfico de faturamento dos últimos 14 dias: total ${money(total)}, melhor dia ${formatDate(best.date)} com ${money(Math.max(0,best.val))}.`);
```

```html
<svg aria-hidden="true" viewBox="0 0 ${w} ${h}" preserveAspectRatio="none">
```

Teste manual: árvore acessível deve anunciar resumo uma vez e omitir o SVG interno; alternar tema mantém gráfico.

### C8 — lembretes horários com ciclo de vida

**Status:** aplicado. Adicionados localReminderInterval e syncLocalReminderTimer; init e concessão de permissão iniciam um único timer3600000ms. visibilitychange limpa o anterior e só rearma se documento visível e permissão granted. A verificação inicial1200ms permanece com guard de visibilidade; falhas de Notification agora registram console.warn.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1,4 @@
-function checkLocalReminders(){if(typeof Notification==='undefined'||Notification.permission!=='granted')return;const t=today(),low=db.ingredients.filter(i=>ingredientStock(i.id)<=num(i.min));if(low.length)try{new Notification('ReceitaPro — estoque',{body:`${low.length} item(ns) chegaram ao nível mínimo.`})}catch(e){}}
+let localReminderInterval=null;
+function syncLocalReminderTimer(){if(localReminderInterval!==null){clearInterval(localReminderInterval);localReminderInterval=null}if(document.visibilityState==='hidden'||typeof Notification==='undefined'||Notification.permission!=='granted')return;localReminderInterval=setInterval(checkLocalReminders,3600000)}
+function checkLocalReminders(){if(document.visibilityState==='hidden'||typeof Notification==='undefined'||Notification.permission!=='granted'){syncLocalReminderTimer();return}const low=db.ingredients.filter(i=>ingredientStock(i.id)<=num(i.min));if(low.length)try{new Notification('ReceitaPro — estoque',{body:`${low.length} item(ns) chegaram ao nível mínimo.`})}catch(e){console.warn('Falha ao mostrar lembrete de estoque',e)}}
+document.addEventListener('visibilitychange',syncLocalReminderTimer);
```

Teste manual: permitir notificações, confirmar um único timer horário; esconder aba remove timer; voltar rearma somente um; callback deve notificar insumos no mínimo. Sem permissão nenhum timer é criado.

### C9 — auditoria final do bloco

**Status:** aplicado. Substituído o único token antigo indefinido `var(--line)` por `var(--color-border)` na separação da configuração da nuvem. Datas de negócio já usam hoje/localISO; nenhum sliceUTC0,10. Native alert/confirm existem somente nos dois fallbacks de ui. Todos os atributos onclick restantes são chamadas simples expostas.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-style="height:1px;background:var(--line);margin:14px 0"
+style="height:1px;background:var(--color-border);margin:14px 0"
```

Contagens:

- Native alert/confirm fora de fallback ui: 0.
- Native calls dentro de fallback ui: 2.
- Datas de negócio `.toISOString().slice(0,10)`: 0.
- Tokens antigos/indefinidos: 1 → 0.
- onclick complexo: 19 → 0.
- onclick com arrow/entity lógica: 1 → 0.
- Propriedades JS `.onclick=`: 0 após conversão.
- Chamadas inline simples restantes: 67.

Teste manual: abrir/configurar nuvem e verificar linha divisória; repetir fluxos de validação e navegação, conferindo console.

### Validação e checkpoint

- node --check: aprovado.
- Estática C1–C9: aprovada; evidência stage4-static.json.
- Sidebar segue uma única definição global e paletas/contraste de B preservados.
- Exposed recebeu somente os seis helpers válidos; nenhuma função anterior removida/renomeada.
- Cada handler convertido mantém equivalente delegado/addEventListener; zero eval/new Function.
- Browser checkpoint de foco/inputmode/autocomplete/dialogs/print/ARIA/reminders: passou (14 verificações C e fluxo a–j repetido).
- Nenhuma alteração de Etapa5 ou schema nesta edição.


## Etapa 5 — itens de UX integrados

Todos os itens 5.1–5.20 foram aplicados; 5.15 era opcional e também foi incluído. Toda a implementação dos trechos abaixo está incorporada no HTML.


### 5.1 — aplicado

Funções: navigationKeydown62, cancelOpenDialog62, submitActiveForm62, initNavigation62. Dois listeners duplicados viram um handler central, preservando Ctrl/Meta+K,/,AltV/P/L,CtrlZ e?. Novos AltD/I/R/S/F/A/N/B/C e Ctrl/MetaS estão documentados em Atalhos. Escape dispara cancel antes de fechar, resolvendo ui.confirm; requestSubmit só usa formulário visível na página/diálogo ativo. AltGr/composição/repetição não disparam ações involuntárias.

Manual: navegar por cada atalho; cancelar exclusão com Escape e abrir nova confirmação; preencher compra válida e CtrlS; com busca aberta CtrlS não submete página de fundo.

ANTES (trecho real):

```javascript
  document.addEventListener('keydown',e=>{const tag=document.activeElement?.tagName,editing=['INPUT','TEXTAREA','SELECT'].includes(tag);let k=e.key.toLowerCase();if(k==='?'&&!editing){e.preventDefault();openShortcutHelp();return}if(!(e.altKey||e.ctrlKey))return;if(e.ctrlKey&&k==='k'){e.preventDefault();openGlobalSearch()}else if(e.altKey&&k==='v'){e.preventDefault();goPage('sales');focusId('saleQty')}else if(e.altKey&&k==='p'){e.preventDefault();goPage('production');setProductionTab('register');focusId('prodQty')}else if(e.altKey&&k==='l'){e.preventDefault();goPage('production');setProductionTab('planner');focusId('planQty')}else if(e.ctrlKey&&k==='z'&&!e.shiftKey&&!editing){e.preventDefault();undoLast()}});
```

DEPOIS (trecho real):

```javascript
function cancelOpenDialog62(){
  const dialogs=[...document.querySelectorAll('dialog[open]')];
  const dialog=document.activeElement?.closest('dialog[open]')||dialogs.at(-1);
  if(!dialog)return false;
  const allowed=dialog.dispatchEvent(new Event('cancel',{cancelable:true}));
  if(allowed&&dialog.open)dialog.close();
  return true;
}

function submitActiveForm62(){
  const active=document.activeElement,dialog=active?.closest('dialog[open]')||[...document.querySelectorAll('dialog[open]')].at(-1);
  const scope=dialog||document.querySelector('.page.active');
  const visible=element=>!!element&&element.getClientRects().length>0&&!element.closest('[hidden],.hide,.wizard-hidden')&&getComputedStyle(element).visibility!=='hidden';
  const focused=active?.closest('form');
  const form=visible(focused)&&(!dialog||dialog.contains(focused))?focused:[...(scope?.querySelectorAll('form')||[])].find(visible);
  if(!form)return;
  const submit=[...form.querySelectorAll('button[type="submit"],input[type="submit"]')].find(button=>visible(button)&&!button.disabled);
  if(submit)form.requestSubmit(submit);else form.requestSubmit();
}
function navigationKeydown62(event){
  if(event.defaultPrevented||event.isComposing)return;
  const key=event.key.toLowerCase(),active=document.activeElement;
  const editing=!!active?.closest('input,textarea,select,[contenteditable="true"]');
  const searching=$('searchDialog')?.open&&active===$('globalSearchInput');
  if(searching&&['arrowdown','arrowup','enter','home','end'].includes(key)){
    const buttons=[...$('globalSearchResults').querySelectorAll('[data-search-action-key]')];
    if(!buttons.length){if(key==='enter')event.preventDefault();return}
    event.preventDefault();
    if(key==='enter'){currentSearchActions.get(buttons[navigation62.selected]?.dataset.searchActionKey||buttons[0].dataset.searchActionKey)?.();return}
    if(key==='home')navigation62.selected=0;
    else if(key==='end')navigation62.selected=buttons.length-1;
    else navigation62.selected=(navigation62.selected+(key==='arrowdown'?1:-1)+buttons.length)%buttons.length;
    selectPaletteResult62();return;
  }
  if(key==='escape'){
    if(!$('fabRadial62')?.hidden){event.preventDefault();closeFabRadial62(true);return}
    if(cancelOpenDialog62())event.preventDefault();return;
  }
  if(event.ctrlKey&&event.altKey)return; // AltGr é entrada de texto, não um atalho de navegação.
  const command=event.ctrlKey||event.metaKey;
  if(command&&key==='s'){event.preventDefault();if(!event.repeat)submitActiveForm62();return}
  if(command&&key==='k'){event.preventDefault();if(!event.repeat)openGlobalSearch();return}
  if(event.ctrlKey&&key==='z'&&!event.shiftKey&&!editing){event.preventDefault();if(!event.repeat)undoLast();return}
  if(key==='?'&&!editing&&!command&&!event.altKey){event.preventDefault();if(!event.repeat)openShortcutHelp();return}
  if(key==='/'&&!editing&&!command&&!event.altKey){event.preventDefault();if(!event.repeat)openGlobalSearch();return}
  if(!event.altKey||event.metaKey||event.shiftKey)return;
  const actions={d:()=>goPage('dashboard'),i:()=>goPage('inventory'),r:()=>goPage('recipes'),s:()=>goPage('sales'),f:()=>goPage('finance'),a:()=>goPage('settings'),n:()=>openPOS(),b:()=>openGlobalSearch(),c:()=>openQuickAudit62(),v:()=>{goPage('sales');focusId('saleQty')},p:()=>{goPage('production');setProductionTab('register');focusId('prodQty')},l:()=>{goPage('production');setProductionTab('planner');focusId('planQty')}};
  if(actions[key]){event.preventDefault();if(!event.repeat)actions[key]()}
}
```



### 5.2 — aplicado

Funções: renderPrefixedSearch62, enhancePaletteResults62, selectPaletteResult62; wrappers renderGlobalSearch/openGlobalSearch. Sem prefixo chama busca anterior com todos os grupos. > só navegação; @ agrupa clientes fiado pendente; # produtos/receitas. Maps de chaves únicas guardam ações. Combobox/listbox/options têm aria-activedescendant/selected; setas selecionam, Enter executa, Escape fecha.

Manual: CtrlK→>vendas→Enter; @ omite clientes já pagos; #bolo abre edição; busca Leite continua mostrando insumos/receitas/compras; setas/Enter mantêm foco no input.

ANTES (trecho real):

```javascript
function renderGlobalSearch(q){const k=keyName(q),root=$('globalSearchResults');if(!root)return;currentSearchActions.clear();if(!k){root.innerHTML='<div class="empty">Digite um nome, ingrediente, cliente ou valor para buscar.</div>';return}const groups=[];const add=(title,items)=>{if(items.length)groups.push({title,items:items.slice(0,10)})};add('Insumos',db.ingredients.filter(i=>keyName(i.name).includes(k)).map(i=>({title:i.name,meta:`Saldo ${qtyText(ingredientStock(i.id),i.unit)} • ${money(i.price)} / ${qtyText(i.packQty,i.unit)}`,run:()=>{closeSearchDialog();goPage('inventory');setInventoryTab('stock')}})));add('Receitas',db.recipes.filter(r=>keyName(r.name).includes(k)||(r.items||[]).some(it=>keyName(ingredient(it.ingredientId)?.name).includes(k))).map(r=>({title:r.name,meta:`Custo ${money(unitCost(r))}/un. • venda ${money(r.price)} • prontos ${readyStock(r.id)}`,run:()=>{closeSearchDialog();goPage('recipes');editRecipe(r.id)}})));add('Compras',db.purchases.filter(p=>keyName(ingredient(p.ingredientId)?.name||p.ingredientName).includes(k)||keyName(p.note).includes(k)||keyName(p.batch).includes(k)).map(p=>({title:ingredient(p.ingredientId)?.name||p.ingredientName||'Compra',meta:`${formatDate(p.date)} • ${qtyText(p.qty,p.unit)} • ${money(p.value)}`,run:()=>{closeSearchDialog();editPurchase(p.id)}})));add('Produções',db.production.filter(p=>keyName(recipe(p.recipeId)?.name).includes(k)||keyName(p.note).includes(k)).map(p=>({title:recipe(p.recipeId)?.name||'Produto removido',meta:`${formatDate(p.date)} • ${p.qty-p.waste} un. prontas`,run:()=>{closeSearchDialog();goPage('production')}})));add('Vendas e fiados',db.sales.filter(s=>keyName(recipe(s.recipeId)?.name).includes(k)||keyName(s.customer).includes(k)||keyName(s.payment).includes(k)||keyName(s.note).includes(k)).map(s=>({title:recipe(s.recipeId)?.name||'Produto removido',meta:`${formatDate(s.date)} • ${s.qty} un. • ${money(s.qty*s.price)} • ${s.payment}${s.customer?' • '+s.customer:''}`,run:()=>{closeSearchDialog();goPage('sales')}})));add('Saídas sem venda',db.productAdjustments.filter(a=>keyName(recipe(a.recipeId)?.name).includes(k)||keyName(a.reason).includes(k)||keyName(a.note).includes(k)).map(a=>({title:recipe(a.recipeId)?.name||'Produto removido',meta:`${formatDate(a.date)} • ${a.qty} un. • ${a.reason}`,run:()=>{closeSearchDialog();goPage('sales')}})));add('Despesas',db.expenses.filter(e=>keyName(e.name).includes(k)||keyName(e.category).includes(k)).map(e=>({title:e.name,meta:`${formatDate(e.date)} • ${e.category} • ${money(e.value)}`,run:()=>{closeSearchDialog();goPage('finance')}})));root.innerHTML=groups.length?groups.map(g=>`<section class="search-group"><h4>${esc(g.title)}</h4>${g.items.map(r=>{const key=`search-${++uiActionKeySequence}`;currentSearchActions.set(key,r.run);return `<button class="search-result" type="button" data-search-action-key="${key}"><b>${esc(r.title)}</b><span>${esc(r.meta)}</span></button>`}).join('')}</section>`).join(''):'<div class="empty">Nada encontrado.</div>'}
```

DEPOIS (trecho real):

```javascript
function renderPrefixedSearch62(query){
  const prefix=query.trimStart()[0],term=keyName(query.trimStart().slice(1)),root=$('globalSearchResults');
  const actions=new Map();
  const add=(key,title,meta,run,keywords=title)=>{if(!term||keyName(keywords).includes(term))actions.set(key,{title,meta,run})};
  if(prefix==='>'){
    const destinations=[['dashboard','Meu dia','Dashboard'],['inventory','Estoque','Insumos inventário'],['purchases','Compras','Estoque compras'],['adjustments','Contagem / ajustes','Estoque contagem ajustes'],['recipes','Receitas','Produtos receitas'],['production','Produção','Produção registro'],['planner','Planejador','Produção planejador'],['sales','Vendas','Vendas saídas fiados'],['finance','Financeiro','Finanças carteira'],['settings','Ajustes','Ajustes configurações']];
    for(const [id,title,keywords] of destinations)add(`navigation62-${id}`,title,'Abrir seção',()=>{
      closeSearchDialog();
      if(id==='purchases'||id==='adjustments'){goPage('inventory');setInventoryTab(id)}
      else if(id==='planner'){goPage('production');setProductionTab('planner')}
      else goPage(id);
    },title+' '+keywords);
  }else if(prefix==='@'){
    const customers=new Map();
    for(const sale of db.sales.filter(s=>s.payment==='Fiado'&&!s.paid)){
      const name=sale.customer.trim()||'Cliente sem nome',key=keyName(name);
      if(!customers.has(key))customers.set(key,{name,sales:[]});customers.get(key).sales.push(sale);
    }
    for(const [key,customer] of customers){
      const value=customer.sales.reduce((sum,sale)=>sum+num(sale.qty)*num(sale.price),0);
      add(`customer62-${key}`,customer.name,`${customer.sales.length} fiado(s) pendente(s) • ${money(value)}`,()=>{closeSearchDialog();goPage('sales');$('receivableList')?.scrollIntoView({block:'center',behavior:'smooth'})});
    }
  }else if(prefix==='#'){
    for(const item of db.recipes)add(`product62-${item.id}`,item.name,`Venda ${money(item.price)} • ${readyStock(item.id)} un. prontas`,()=>{closeSearchDialog();editRecipe(item.id)});
  }
  currentSearchActions.clear();
  root.innerHTML=actions.size?[...actions].map(([key,action])=>{
    currentSearchActions.set(key,action.run);
    return `<button class="search-result" type="button" data-search-action-key="${esc(key)}"><b>${esc(action.title)}</b><span>${esc(action.meta)}</span></button>`;
  }).join(''):'<div class="empty">Nada encontrado.</div>';
}
```



### 5.3 — Enter para editar/contar

Status: aplicado. Funções/arquivo: renderTable; rowMainAction62; editClosing62; editProductAdjustment62; previewLoss; lossForm; renderPlan; delegatedActions e exposed.

ANTES (trecho real):

```js
function renderTable(tbodyId,html){const tbody=$(tbodyId);if(!tbody)return;tbody.innerHTML=html;const table=tbody.closest('table[data-card-view="true"]');if(table)applyTableLabels(table)}
```

DEPOIS (trecho real):

```js
function rowMainAction62(id,row){
  const edit=row.querySelector('[data-action^="edit"],[data-action="startCount"]');
  if(edit)return ()=>edit.click();
  if(id==='adjustRows'){const record=tableRecord62(id,row);if(record)return ()=>startCount(record.ingredientId)}
  if(id==='closingRows'){const record=tableRecord62(id,row);if(record)return ()=>editClosing62(record.id)}
  if(id==='lossRows'){const record=tableRecord62(id,row);if(record)return ()=>editProductAdjustment62(record.id)}
  if(id==='planRows'&&row.dataset.ingredientId)return ()=>editIngredient(row.dataset.ingredientId);
  return null;
}
function editClosing62(id){const record=db.cashClosings.find(x=>x.id===id);if(!record)return;goPage('finance');for(const [field,key] of [['closingDate','date'],['closingOpening','opening'],['closingCashOut','cashOut'],['closingCounted','counted'],['closingNote','note']])$(field).value=record[key]??'';renderClosingPreview();focusId('closingOpening')}
function editProductAdjustment62(id){const record=db.productAdjustments.find(x=>x.id===id);if(!record)return;goPage('sales');openLossDialog(record.recipeId);$('lossId62').value=record.id;$('lossQty').value=record.qty;$('lossReason').value=record.reason;$('lossDate').value=record.date;$('lossNote').value=record.note||'';$('lossDialogTitleA11y').textContent='Editar saída sem venda';$('lossForm').querySelector('button[type="submit"]').textContent='Salvar correção';previewLoss();scheduleTableEnhancements62();focusId(comboStates62.get('lossRecipe')?.input.id||'lossQty')}
```

Linhas reais recebem tabindex=0 e role=row. Enter aciona os editores existentes; ajustes abrem contagem. Fechamentos reutilizam o formulário com a mesma data. Perdas passam a ter edição pelo mesmo id, mantendo custo histórico e devolvendo a quantidade antiga ao limite disponível antes de validar; excedentes continuam pedindo confirmação. Planejador identifica o ingrediente por id e abre seu editor. Botões Editar/Excluir existentes permanecem e Editar também foi acrescentado a perdas/fechamentos.

Teste manual: Foque uma linha com Tab e use Enter; confira id/formulário. Edite uma perda e salve: o número de registros deve ficar igual. Informe uma perda maior que o disponível e cancele a confirmação: o registro deve permanecer. No planejador, Enter deve abrir o insumo correto.



### 5.4 — aplicado

Função: refreshNavigation62 e wrappers de navegação. Linha discreta #contextBreadcrumb62 adicionada acima de #topTitle, cujo título original é preservado. Navegação/subtabs atualizam Estoque/Compras e Produção/Planejador sem persistência adicional.

Manual: alternar Compras/Contagem e Registro/Planejador; verificar contexto e título existente.

ANTES (trecho real):

```javascript
function goPage(page){currentPage=page;$$('.page').forEach(x=>x.classList.toggle('active',x.id===page));$$('#mainNav button').forEach(x=>{const active=x.dataset.page===page;x.classList.toggle('active',active);if(active)x.setAttribute('aria-current','page');else x.removeAttribute('aria-current')});$('topTitle').textContent=PAGE_TITLES[page]||'ReceitaPro';if(page==='recipes'&&!$('recipeId').value&&!document.querySelector('.recipe-line')&&db.ingredients.length)addRecipeItem();renderPage(page);window.scrollTo({top:0,behavior:'smooth'})}
```

DEPOIS (trecho real):

```javascript
function refreshNavigation62(){
  const breadcrumb=$('contextBreadcrumb62');
  if(breadcrumb){
    let text=PAGE_TITLES[currentPage]||'ReceitaPro';
    if(currentPage==='inventory')text='Estoque / '+({stock:'Saldo dos insumos',purchases:'Compras',adjustments:'Contagem / ajustes'}[currentInvTab]||'Estoque');
    if(currentPage==='production')text='Produção / '+({register:'Registro',planner:'Planejador'}[currentProdTab]||'Produção');
    breadcrumb.textContent=text;
  }
```



### 5.5 — aplicado

Função: refreshNavigation62; spans de initNavigation62. Estoque conta saldo estritamente menor que mínimo; Vendas conta registros fiado não pagos; Produção conta receitas prontas>0. Zero fica oculto, aria-label descreve contador, cores#eef8f4/#0b3a2d têm alto contraste. Renders locais/restauração/nuvem atualizam badges sem mudar alertas existentes.

Manual: saldo igual ao mínimo não conta; abaixo conta; receber fiado e esgotar produto pronto atualizam badges. Conferir claro/escuro/mobile.

ANTES (trecho real):

```javascript
<button data-page="inventory"><span class="nav-icon">▦</span><span>Estoque</span></button>
<button data-page="sales"><span class="nav-icon">$</span><span>Vendas</span></button>
<button data-page="production"><span class="nav-icon">◉</span><span>Produção</span></button>
```

DEPOIS (trecho real):

```javascript
  const counts={inventory:db.ingredients.filter(item=>ingredientStock(item.id)<num(item.min)).length,sales:db.sales.filter(sale=>sale.payment==='Fiado'&&!sale.paid).length,production:db.recipes.filter(item=>readyStock(item.id)>0).length};
  const labels={inventory:'insumo(s) abaixo do estoque mínimo',sales:'fiado(s) pendente(s)',production:'produto(s) com estoque pronto'};
  for(const [page,count] of Object.entries(counts)){
    const badge=$(`navigationBadge62-${page}`);if(!badge)continue;
    badge.hidden=count===0;badge.textContent=String(count);badge.setAttribute('aria-label',`${count} ${labels[page]}`);
  }
```



### 5.6 — aplicado

Funções: initFabRadial62, openFabRadial62, closeFabRadial62. Clique/Enter preservam quickDialog; pressionar300ms abre Venda/Produção/Compra/Perda e liberação longa suprime clique seguinte. Menu/menuitem usam setas/foco; cancel/resize/outside/Escape limpam timer/estado. Menu fixed conserva posições no hover; smooth scroll programático não cancela gesto. Contêiner tem área real para visibilidade e botões clicáveis, sem bloquear regiões transparentes.

Manual: clique rápido/Enter; segurar>300ms e usar cada ação; Escape/clique fora; cancelar/rotacionar antes300ms não abre menu atrasado. Iniciar gesto durante smooth scroll de navegação.

ANTES (trecho real):

```javascript
<dialog aria-labelledby="shortcutDialogTitle" aria-modal="true" id="shortcutDialog" role="dialog"><div class="dialog-body"><div class="search-head"><div><h3 id="shortcutDialogTitle">Atalhos do teclado</h3><p>Use estes atalhos para trabalhar mais rápido.</p></div><button aria-label="Fechar" class="btn icon ghost" data-action="closeDialog" data-id="shortcutDialog" type="button">×</button></div><div class="shortcut-list"><div><kbd>Ctrl</kbd> + <kbd>K</kbd><span>Busca global</span></div><div><kbd>Alt</kbd> + <kbd>V</kbd><span>Nova venda</span></div><div><kbd>Alt</kbd> + <kbd>P</kbd><span>Nova produção</span></div><div><kbd>Alt</kbd> + <kbd>L</kbd><span>Planejador</span></div><div><kbd>Ctrl</kbd> + <kbd>Z</kbd><span>Desfazer</span></div><div><kbd>?</kbd><span>Este painel</span></div></div></div></dialog><dialog aria-labelledby="onboardingTitle" aria-modal="true" class="onboarding-dialog" id="onboardingDialog" role="dialog"><div class="dialog-body"><div class="onboarding-progress" id="onboardingProgress"></div><h3 class="visually-hidden" id="onboardingTitle">Introdução ao ReceitaPro</h3><div id="onboardingContent"></div></div><div class="dialog-actions"><button class="btn ghost" id="onboardingBack" type="button">Voltar</button><button class="btn primary" id="onboardingNext" type="button">Próximo</button></div></dialog><dialog aria-labelledby="posDialogTitle" aria-modal="true" class="pos-dialog" id="posDialog" role="dialog"><div class="dialog-body"><div class="search-head"><div><h3 id="posDialogTitle">PDV — venda rápida</h3><p>Toque nos produtos, confira o carrinho e finalize.</p></div><button aria-label="Fechar PDV" class="btn icon ghost" onclick="closePOS()" type="button">×</button></div><div class="pos-layout"><div><input aria-label="Buscar produto no PDV" class="input" id="posSearch" placeholder="Buscar produto..."/><div class="pos-products" id="posProducts"></div></div><div class="pos-cart"><h4>Carrinho</h4><div id="posCartRows"></div><div class="pos-total"><span>Total</span><strong id="posTotal">R$ 0,00</strong></div><div class="form-grid"><label class="field"><span>Pagamento</span><select id="posPayment"><option>Pix</option><option>Dinheiro</option><option>Cartão</option><option>Outro</option></select></label><label class="field"><span>Taxa total (R$)</span><input inputmode="decimal" class="input" id="posFee" min="0" step="0.01" type="number" value="0"/></label></div><div class="form-actions"><button class="btn ghost" onclick="clearPOSCart()" type="button">Limpar</button><button class="btn primary" id="posFinish" onclick="finishPOS()" type="button">Finalizar venda</button></div></div></div></div></dialog><section aria-hidden="true" class="print-report" id="printReport"></section><button aria-label="Ação rápida" class="fab" onclick="openQuickAction()" type="button">+</button>
```

DEPOIS (trecho real):

```javascript
function closeFabRadial62(restoreFocus=false){
  clearTimeout(navigation62.longPressTimer);navigation62.longPressTimer=null;navigation62.pointerId=null;
  const menu=$('fabRadial62'),fab=document.querySelector('.fab');
  const focusedWithin=menu?.contains(document.activeElement);
  if(menu)menu.hidden=true;fab?.setAttribute('aria-expanded','false');
  if(restoreFocus||focusedWithin)fab?.focus();
}

function openFabRadial62(){
  const fab=document.querySelector('.fab'),menu=$('fabRadial62'),rect=fab.getBoundingClientRect();
  menu.style.left=`${Math.min(rect.left+rect.width/2,innerWidth-50)-192}px`;menu.style.top=`${rect.top+rect.height/2-166}px`;menu.hidden=false;
  fab.setAttribute('aria-expanded','true');menu.querySelector('button')?.focus({preventScroll:true});
}

function initFabRadial62(){
  const fab=document.querySelector('.fab');if(!fab)return;
  fab.removeAttribute('onclick');fab.setAttribute('aria-controls','fabRadial62');fab.setAttribute('aria-expanded','false');fab.setAttribute('aria-haspopup','menu');
  const menu=document.createElement('div');menu.id='fabRadial62';menu.className='fab-radial62';menu.hidden=true;menu.setAttribute('role','menu');menu.setAttribute('aria-label','Ações rápidas');
  const actions=[['sale','Venda',()=>quickGo('sale')],['production','Produção',()=>quickGo('production')],['purchase','Compra',()=>quickGo('purchase')],['loss','Perda',()=>openLossDialog()]];
  actions.forEach(([key,label,run],index)=>{
    const button=document.createElement('button');button.type='button';button.className='btn soft';button.dataset.radialAction62=key;button.setAttribute('role','menuitem');button.textContent=label;button.style.setProperty('--radial-x62',[-144,-125,-72,0][index]+'px');button.style.setProperty('--radial-y62',[0,-72,-125,-144][index]+'px');
    button.addEventListener('click',()=>{closeFabRadial62();run()});menu.appendChild(button);
  });
  document.body.appendChild(menu);
  const resetPointer=()=>{clearTimeout(navigation62.longPressTimer);navigation62.longPressTimer=null;navigation62.pointerId=null};
  fab.addEventListener('pointerdown',event=>{
    if(event.button!==0)return;resetPointer();navigation62.longPressed=false;navigation62.pointerId=event.pointerId;
    navigation62.longPressTimer=setTimeout(()=>{navigation62.longPressTimer=null;navigation62.longPressed=true;openFabRadial62()},300);
  });
  document.addEventListener('pointerup',event=>{if(event.pointerId===navigation62.pointerId)resetPointer()});
  fab.addEventListener('pointercancel',()=>{resetPointer();navigation62.longPressed=false;closeFabRadial62()});
  fab.addEventListener('pointerleave',()=>{if(navigation62.longPressTimer)resetPointer()});
  fab.addEventListener('contextmenu',event=>{if(navigation62.longPressed)event.preventDefault()});
  fab.addEventListener('click',event=>{
    resetPointer();
    if(navigation62.longPressed&&event.detail!==0){navigation62.longPressed=false;event.preventDefault();return}
    navigation62.longPressed=false;closeFabRadial62();openQuickAction();
  });
  document.addEventListener('pointerdown',event=>{if(!menu.hidden&&!menu.contains(event.target)&&!fab.contains(event.target))closeFabRadial62()});
  menu.addEventListener('keydown',event=>{
    if(!['ArrowDown','ArrowUp','ArrowLeft','ArrowRight','Home','End'].includes(event.key))return;
    event.preventDefault();const buttons=[...menu.querySelectorAll('button')],index=buttons.indexOf(document.activeElement);
    const next=event.key==='Home'?0:event.key==='End'?buttons.length-1:(index+(['ArrowDown','ArrowRight'].includes(event.key)?1:-1)+buttons.length)%buttons.length;
    buttons[next].focus();
  });
  window.addEventListener('resize',()=>closeFabRadial62());
}
```



### 5.7 — Filtro nativo para listas grandes

Status: aplicado. Funções/arquivo: enhanceCombo62; renderSelectors wrapper; setControlError wrapper; handlers input/reset/submit/invalid.

ANTES (trecho real):

```js
function renderSelectors(){let ios=ingredientOptions(),ros=recipeOptions();['purchaseIngredient','adjustIngredient'].forEach(id=>updateSelect(id,ios,'Cadastre um insumo'));['prodRecipe','saleRecipe','planRecipe','lossRecipe'].forEach(id=>updateSelect(id,ros,'Cadastre uma receita'));let sr=recipe($('saleRecipe')?.value);if(sr&&!$('saleId')?.value&&!$('salePrice')?.value)$('salePrice').value=sr.price;updatePurchaseHint();updateAdjustPreview();previewProduction();previewSale();renderPlan()}
```

DEPOIS (trecho real):

```js
function enhanceCombo62(id){
  const select=$(id);if(!select)return;
  const options=[...select.options].filter(option=>option.value&&!option.disabled);
  let state=comboStates62.get(id);
  if(!state&&options.length>15){
    const wrap=document.createElement('div');wrap.className='rp-combo62';
    const input=document.createElement('input');input.id=id+'Combo62';input.type='text';input.className='input';input.autocomplete='off';input.placeholder='Filtrar e escolher…';input.setAttribute('aria-label',select.getAttribute('aria-label')||select.closest('label')?.querySelector('span')?.textContent||'Filtrar opções');
    const list=document.createElement('datalist');list.id=id+'Choices62';input.setAttribute('list',list.id);
    const hint=document.createElement('span');hint.id=id+'ComboHint62';hint.className='field-help';hint.textContent='Digite para filtrar e selecione uma opção da lista.';input.setAttribute('aria-describedby',[select.getAttribute('aria-describedby'),hint.id].filter(Boolean).join(' '));
    wrap.append(input,list,hint);select.before(wrap);
    state={input,list,wrap,required:select.required,choices:new Map(),signature:'',invalid:false,lastValue:select.value};comboStates62.set(id,state);
    select.form?.addEventListener('reset',()=>{state.invalid=false;state.input.value='';state.input.setCustomValidity('');state.lastValue='';clearFieldErrors(select.form);scheduleTableEnhancements62()});
    input.addEventListener('input',()=>{const value=state.choices.get(input.value);state.invalid=!value;select.value=value||'';state.lastValue=select.value;input.setCustomValidity(value?'':'Escolha uma opção válida da lista.');if(value){input.removeAttribute('aria-invalid');const err=$(input.id+'Error');err?.remove();input.setAttribute('aria-describedby',(input.getAttribute('aria-describedby')||'').split(/\s+/).filter(x=>x!==input.id+'Error').join(' '))}select.dispatchEvent(new Event('change',{bubbles:true}))});
    select.addEventListener('change',()=>{if(!state.invalid){const match=[...state.choices].find(([,value])=>value===select.value);input.value=match?.[0]||'';input.setCustomValidity(select.value?'':'Escolha uma opção válida da lista.');state.lastValue=select.value}});
  }
  if(!state)return;
  const active=options.length>15;state.wrap.hidden=!active;state.input.disabled=!active;select.hidden=active;select.required=active?false:state.required;state.input.required=active&&state.required;
  if(!active)return;
  const counts=new Map();for(const option of options)counts.set(option.textContent,(counts.get(option.textContent)||0)+1);
  const choices=options.map(option=>[option.textContent+(counts.get(option.textContent)>1?' ['+option.value+']':''),option.value]);
  const signature=JSON.stringify(choices);
  if(signature!==state.signature){state.signature=signature;state.choices=new Map(choices);state.list.replaceChildren(...choices.map(([label,value])=>{const option=document.createElement('option');option.value=label;option.dataset.id=value;return option}))}
  if(state.invalid&&select.value===state.lastValue){select.value='';state.input.setCustomValidity('Escolha uma opção válida da lista.');return}
  const match=choices.find(([,value])=>value===select.value);
  state.input.value=match?.[0]||'';state.input.setCustomValidity(select.value?'':'Escolha uma opção válida da lista.');state.invalid=!select.value;state.lastValue=select.value;
}
```

Somente listas acima de 15 opções ganham input[list]/datalist nativo com rótulo e ajuda. A seleção continua usando o id original. Texto sem correspondência limpa o select oculto e fica inválido, com erro/foco no campo visível inclusive em forms novalidate. Cancelar/resetar limpa query e invalidade; listas pequenas continuam com select original.

Teste manual: Com 18 insumos/produtos, escolha uma opção da lista e observe preview original. Digite um nome inexistente e tente salvar: nenhum registro deve ser criado com a seleção antiga. Cancele/reset o formulário: uma seleção válida deve ser recuperada. Com <=15 opções deve aparecer apenas o select.



### 5.8 — ações reversíveis com Desfazer

Funções: commit, deleteIngredient/Purchase/Adjustment/Recipe/Production/Closing/Sale/ProductAdjustment/Expense/WalletMovement e toastStack62. A confirmação das dez exclusões reversíveis foi substituída por commit direto e toast com Desfazer. As restrições de histórico para ingredientes/receitas continuam; clearAll e restaurações permanecem com confirmação.

O botão de cada toast guarda a identidade do seu undoEntry. Só fica ativo quando esse snapshot é o topo da pilha; alterações novas desabilitam os anteriores e uma guarda no callback impede desfazer outro commit. A ordem original/limite20 da pilha é preservada. Quando a produção excluída torna saldo negativo, um toast adicional mantém o aviso.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function commit(label,mutator){saveAutoBackup(label);undoStack.push({label,data:deepClone(db)});if(undoStack.length>20)undoStack.shift();mutator();db=migrateData(db);db.activityLog.push({id:uid(),ts:new Date().toISOString(),label,page:currentPage,actor:db.operatorName||'Você',createdAt:new Date().toISOString()});if(db.activityLog.length>300)db.activityLog=db.activityLog.slice(-300);persist();toast(label);updateUndo()}
+function commit(label,mutator){saveAutoBackup(label);const undoEntry={id:uid(),label,data:deepClone(db)};undoStack.push(undoEntry);if(undoStack.length>20)undoStack.shift();mutator();db=migrateData(db);db.activityLog.push({id:uid(),ts:new Date().toISOString(),label,page:currentPage,actor:db.operatorName||'Você',createdAt:new Date().toISOString()});if(db.activityLog.length>300)db.activityLog=db.activityLog.slice(-300);persist();toast(label,{undoEntry});updateUndo()}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteIngredient(id){let i=ingredient(id);if(!i)return;let usedRecipe=db.recipes.some(r=>(r.items||[]).some(it=>it.ingredientId===id)),used=db.purchases.some(p=>p.ingredientId===id)||db.stockAdjustments.some(a=>a.ingredientId===id)||db.production.some(p=>(p.consumedItems||[]).some(it=>it.ingredientId===id));if(usedRecipe||used)return ui.alert('Este insumo já está ligado a receitas ou movimentações. Para preservar o histórico, não é possível excluí-lo. Você pode renomeá-lo para “INATIVO — ...” se não quiser mais usar.');confirmAction('Excluir insumo?',`${i.name} será removido.`,()=>commit('Insumo excluído',()=>db.ingredients=db.ingredients.filter(x=>x.id!==id)))}
+function deleteIngredient(id){let i=ingredient(id);if(!i)return;let usedRecipe=db.recipes.some(r=>(r.items||[]).some(it=>it.ingredientId===id)),used=db.purchases.some(p=>p.ingredientId===id)||db.stockAdjustments.some(a=>a.ingredientId===id)||db.production.some(p=>(p.consumedItems||[]).some(it=>it.ingredientId===id));if(usedRecipe||used)return ui.alert('Este insumo já está ligado a receitas ou movimentações. Para preservar o histórico, não é possível excluí-lo. Você pode renomeá-lo para “INATIVO — ...” se não quiser mais usar.');commit('Insumo excluído',()=>db.ingredients=db.ingredients.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deletePurchase(id){let p=purchase(id),i=p&&ingredient(p.ingredientId);if(!p)return;confirmAction('Excluir compra?',`${i?.name||p.ingredientName||'Insumo'}: ${qtyText(p.qty,i?.unit||p.unit||'')} por ${money(p.value)}. O saldo será recalculado automaticamente.`,()=>commit('Compra excluída',()=>db.purchases=db.purchases.filter(x=>x.id!==id)))}
+function deletePurchase(id){let p=purchase(id),i=p&&ingredient(p.ingredientId);if(!p)return;commit('Compra excluída',()=>db.purchases=db.purchases.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteAdjustment(id){let a=db.stockAdjustments.find(x=>x.id===id);if(!a)return;confirmAction('Excluir ajuste?',`O ajuste ${a.delta>=0?'+':''}${qtyText(a.delta,ingredient(a.ingredientId)?.unit||'')} será removido e o saldo recalculado.`,()=>commit('Ajuste excluído',()=>db.stockAdjustments=db.stockAdjustments.filter(x=>x.id!==id)))}
+function deleteAdjustment(id){let a=db.stockAdjustments.find(x=>x.id===id);if(!a)return;commit('Ajuste excluído',()=>db.stockAdjustments=db.stockAdjustments.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteRecipe(id){let r=recipe(id);if(!r)return;let used=db.production.some(p=>p.recipeId===id)||db.sales.some(s=>s.recipeId===id);if(used)return ui.alert('Esse produto já possui produção ou venda no histórico. Para preservar os números antigos, não é possível excluí-lo. Você pode renomeá-lo para “INATIVO — ...”.');confirmAction('Excluir receita?',`${r.name} será removida.`,()=>commit('Receita excluída',()=>db.recipes=db.recipes.filter(x=>x.id!==id)))}
+function deleteRecipe(id){let r=recipe(id);if(!r)return;let used=db.production.some(p=>p.recipeId===id)||db.sales.some(s=>s.recipeId===id);if(used)return ui.alert('Esse produto já possui produção ou venda no histórico. Para preservar os números antigos, não é possível excluí-lo. Você pode renomeá-lo para “INATIVO — ...”.');commit('Receita excluída',()=>db.recipes=db.recipes.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteProduction(id){let p=db.production.find(x=>x.id===id);if(!p)return;let after=readyStock(p.recipeId)-(p.qty-p.waste),warn=after<0?` Isso deixará o estoque pronto em ${after} unidade(s), pois já existem vendas registradas.`:'';confirmAction('Excluir produção?',`Os insumos consumidos voltarão automaticamente ao saldo.${warn}`,()=>commit('Produção excluída',()=>db.production=db.production.filter(x=>x.id!==id)))}
+function deleteProduction(id){let p=db.production.find(x=>x.id===id);if(!p)return;let after=readyStock(p.recipeId)-(p.qty-p.waste),warn=after<0?` Isso deixará o estoque pronto em ${after} unidade(s), pois já existem vendas registradas.`:'';commit('Produção excluída',()=>db.production=db.production.filter(x=>x.id!==id));if(after<0)toast(`Estoque pronto ficou em ${after} unidade(s).`,{undoEntry:undoStack[undoStack.length-1]})}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteClosing(id){let c=db.cashClosings.find(x=>x.id===id);if(!c)return;confirmAction('Excluir fechamento?',`O fechamento de ${formatDate(c.date)} será removido.`,()=>commit('Fechamento excluído',()=>db.cashClosings=db.cashClosings.filter(x=>x.id!==id)))}
+function deleteClosing(id){let c=db.cashClosings.find(x=>x.id===id);if(!c)return;commit('Fechamento excluído',()=>db.cashClosings=db.cashClosings.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteSale(id){let s=db.sales.find(x=>x.id===id);if(!s)return;confirmAction('Excluir venda?',`A venda de ${s.qty} unidade(s), total ${money(s.qty*s.price)}, será removida e o estoque pronto voltará automaticamente.`,()=>commit('Venda excluída',()=>db.sales=db.sales.filter(x=>x.id!==id)))}
+function deleteSale(id){let s=db.sales.find(x=>x.id===id);if(!s)return;commit('Venda excluída',()=>db.sales=db.sales.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteProductAdjustment(id){let a=db.productAdjustments.find(x=>x.id===id);if(!a)return;confirmAction('Excluir esta saída?',`${a.qty} unidade(s) voltarão ao estoque pronto.`,()=>commit('Saída sem venda excluída',()=>db.productAdjustments=db.productAdjustments.filter(x=>x.id!==id)))}
+function deleteProductAdjustment(id){let a=db.productAdjustments.find(x=>x.id===id);if(!a)return;commit('Saída sem venda excluída',()=>db.productAdjustments=db.productAdjustments.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteExpense(id){let e=db.expenses.find(x=>x.id===id);if(!e)return;confirmAction('Excluir despesa?',`${e.name} — ${money(e.value)}.`,()=>commit('Despesa excluída',()=>db.expenses=db.expenses.filter(x=>x.id!==id)))}
+function deleteExpense(id){let e=db.expenses.find(x=>x.id===id);if(!e)return;commit('Despesa excluída',()=>db.expenses=db.expenses.filter(x=>x.id!==id))}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function deleteWalletMovement(id){let m=db.cashMovements.find(x=>x.id===id);if(!m)return;confirmAction('Excluir movimentação da carteira?',`${m.reason}: ${m.delta>=0?'+':''}${money(m.delta)}.`,()=>commit('Movimentação da carteira excluída',()=>db.cashMovements=db.cashMovements.filter(x=>x.id!==id)))}
+function deleteWalletMovement(id){let m=db.cashMovements.find(x=>x.id===id);if(!m)return;commit('Movimentação da carteira excluída',()=>db.cashMovements=db.cashMovements.filter(x=>x.id!==id))}
```

Teste manual: apagar uma compra e clicar Desfazer; registrar outra operação após exclusão e confirmar que o Desfazer do toast antigo fica desabilitado; voltar pela pilha de desfazer sem perder commits novos. Ingrediente/receita com histórico continuam bloqueados; apagar todos continua perguntando.



### 5.9 — Busca visual dos históricos

Status: aplicado. Funções/arquivo: setupTableControls62; enhanceTable62; tableStates62.

ANTES (trecho real):

```js
function renderTable(tbodyId,html){const tbody=$(tbodyId);if(!tbody)return;tbody.innerHTML=html;const table=tbody.closest('table[data-card-view="true"]');if(table)applyTableLabels(table)}
```

DEPOIS (trecho real):

```js
if(definition.search){const input=document.createElement('input');input.type='search';input.className='input search';input.dataset.tableSearch=id;input.placeholder=`Buscar em ${definition.name}…`;input.setAttribute('aria-label',`Buscar em ${definition.name}`);input.value=tableState62(id).query;input.addEventListener('input',()=>{const state=tableState62(id);state.query=input.value;state.limit=50;enhanceTables62()});tools.append(input)}
rows.forEach(row=>{row.hidden=!shown.has(row);if(row.hidden)row.style.setProperty('display','none','important');else row.style.removeProperty('display')});
```

Compras, vendas, produção e perdas recebem busca local. A filtragem mantém os mesmos nodes de linha e muda display/hidden; não chama o render original. Há contagem acessível de resultados.

Teste manual: Digite nome, nota ou data no filtro de um histórico. As linhas devem aparecer/desaparecer sem perder referências DOM; limpar o campo deve recuperar os registros.



### 5.10 — Ordenação e colunas acessíveis

Status: aplicado. Funções/arquivo: tableSortValue62; setupTableControls62; enhanceTable62.

ANTES (trecho real):

```js
function renderTable(tbodyId,html){const tbody=$(tbodyId);if(!tbody)return;tbody.innerHTML=html;const table=tbody.closest('table[data-card-view="true"]');if(table)applyTableLabels(table)}
```

DEPOIS (trecho real):

```js
function tableSortValue62(id,row,column){
  const x=tableRecord62(id,row);
  if(x){
    let values;
    switch(id){
      case'ingredientRows':{const stock=ingredientStock(x.id),cost=num(x.price)/Math.max(.001,num(x.packQty));values=[x.name,cost,stock,Math.max(0,stock)*cost];break}
      case'purchaseRows':values=[x.date,ingredient(x.ingredientId)?.name||x.ingredientName||'',x.qty,x.value,x.qty?x.value/x.qty:0];break;
      case'adjustRows':values=[x.date,ingredient(x.ingredientId)?.name||'',x.delta,x.reason];break;
      case'productionRows':values=[x.date,recipe(x.recipeId)?.name||'',x.qty,x.waste,x.unitCostAtProduction];break;
      case'saleRows':values=[x.date,recipe(x.recipeId)?.name||'',x.qty,x.price,x.qty*x.price,x.fee,x.payment];break;
      case'lossRows':values=[x.date,recipe(x.recipeId)?.name||'',x.qty,x.reason,x.qty*num(x.unitCostAtAdjustment||unitCost(recipe(x.recipeId)))];break;
      case'closingRows':{const expected=num(x.opening)+num(x.cashSales)-num(x.cashOut);values=[x.date,x.totalSales,expected,x.counted,num(x.counted)-expected];break}
      case'expenseRows':values=[x.date,x.name,x.category,x.value];break;
    }
    if(values?.[column]!==undefined)return values[column];
  }
  const text=row.cells[column]?.textContent.trim()||'';
  const date=text.match(/^(\d{2})\/(\d{2})\/(\d{4})/);
  if(date)return `${date[3]}-${date[2]}-${date[1]}`;
  const label=row.closest('table')?.querySelectorAll('thead th')[column]?.dataset.tableLabel62||'';
  if(/custo|valor|faturamento|taxa|quantidade|saldo|estoque|produzido|perdas|preço|estimativa|precisa|tem|falta|comprar|ajuste|contado|diferença|vendas|dinheiro/i.test(label)){const match=text.replace(/−/g,'-').match(/-?\d[\d.]*(?:,\d+)?/);if(match)return Number(match[0].replace(/\./g,'').replace(',','.'))}
  return text;
}
const active=sort?.column===index;th.setAttribute('aria-sort',active?(sort.direction==='asc'?'ascending':'descending'):'none');
```

Cabeçalhos viram botões de ordenação asc/desc, com aria-sort e indicador. Datas e valores dos históricos vêm dos registros originais, evitando comparação por moeda formatada; fallback de colunas numéricas lê números brasileiros. Details “Colunas” permite alternar visibilidade, mantendo Ações sempre disponível. A ordem inicial original só muda quando escolhida uma ordenação.

Teste manual: Clique Data e Valor duas vezes; confira ordem de datas e valores como 1, 2 e 10. Abra Colunas e desmarque Quantidade; recarregue e confira preferência mantida. Ações continua disponível.



### 5.11 — skeleton de nuvem lento

Funções: beginCloudFeedback62/endCloudFeedback62/wrapCloudFeedback62/initFeedback62, wrappers encadeáveis de cloudRequest/processCloudQueue/cloudRefresh/cloudOpenWorkspace. Um contador acompanha operações reais e só mostra skeleton após300ms; fim/falha limpa o timer e esconde o indicador. O aside usa pointer-events:none e não altera locks/awaits da persistência local.

```diff
+function showCloudSkeleton62(show){const node=$('cloudSkeleton62');if(node){node.hidden=!show;node.setAttribute('aria-hidden',String(!show))}}
+function beginCloudFeedback62(){cloudFeedbackCount62++;if(cloudFeedbackCount62===1){clearTimeout(cloudSkeletonTimer62);cloudSkeletonTimer62=setTimeout(()=>{cloudSkeletonTimer62=null;if(cloudFeedbackCount62)showCloudSkeleton62(true)},300)}}
+function endCloudFeedback62(){cloudFeedbackCount62=Math.max(0,cloudFeedbackCount62-1);if(!cloudFeedbackCount62){clearTimeout(cloudSkeletonTimer62);cloudSkeletonTimer62=null;showCloudSkeleton62(false)}}
+function wrapCloudFeedback62(fn){return async function(...args){beginCloudFeedback62();try{return await fn.apply(this,args)}finally{endCloudFeedback62()}}}
```

Teste manual: atrasar RPC por500ms; antes300ms skeleton não aparece, depois aparece; cadastrar/alterar dado enquanto RPC aguarda e confirmar resposta imediata/persistência; ao terminar ou falhar, indicador some.



### 5.12 — Preferências locais por usuário

Status: aplicado. Funções/arquivo: SCHEMA_VERSION=12; emptyDB; migrateData; saveUiPreferences62; restore/applyUiPreferences62; normalizedData/localPersist/cloudView/goPage/setPeriod wrappers.

ANTES (trecho real):

```js
const APP_DB_KEY='receitaProDB', OLD_DB_KEY='geladinhoProDB', THEME_KEY='receitaProTheme', AUTO_BACKUP_KEY='receitaProAutoBackups', SCHEMA_VERSION=11;
function emptyDB(){return {version:SCHEMA_VERSION,ingredients:[],recipes:[],production:[],sales:[],expenses:[],purchases:[],stockAdjustments:[],productAdjustments:[],cashClosings:[],cashMovements:[],activityLog:[],cashAccount:{configured:false,openingBalance:0,startedAt:'',mode:'from_now'},reinvestPct:50,businessName:'',targetMargin:60,monthlyGoal:0,defaultPayment:'Pix',uiMode:'beginner',fontScale:'large',operatorName:'',updatedAt:new Date().toISOString()}}
```

DEPOIS (trecho real):

```js
function defaultUiPreferences62(){return {columns:{},sort:{},lastPage:'dashboard',lastPeriod:'today',from:'',to:''}}
function normalizeUiPreferences62(value){
  const source=value&&typeof value==='object'&&!Array.isArray(value)?deepClone(value):{};
  const prefs={...defaultUiPreferences62(),...source};
  prefs.columns=source.columns&&typeof source.columns==='object'&&!Array.isArray(source.columns)?source.columns:{};
  prefs.sort=source.sort&&typeof source.sort==='object'&&!Array.isArray(source.sort)?source.sort:{};
  prefs.lastPage=['dashboard','inventory','recipes','production','sales','finance','settings'].includes(source.lastPage)?source.lastPage:'dashboard';
  prefs.lastPeriod=['today','7d','month','all','custom'].includes(source.lastPeriod)?source.lastPeriod:'today';
  for(const key of ['from','to'])prefs[key]=/^\d{4}-\d{2}-\d{2}$/.test(String(source[key]||''))?String(source[key]):'';
  return prefs;
}
let uiPreferenceOwner62=null,localUiPreferences62=null,restoringUiPreferences62=false,uiPreferencesLoaded62=false;
function saveUiPreferences62(patch={}){
  if(!uiPreferencesLoaded62)return;
  ensureUiPreferenceOwner62();
  const {columns,sort,lastPage,lastPeriod,from,to,...metadata}=db.uiPrefs||{};
  localUiPreferences62=normalizeUiPreferences62({...localUiPreferences62,...metadata,...patch});
  db.uiPrefs=deepClone(localUiPreferences62);
  safeLocalSet(uiPreferenceStorageKey62(),JSON.stringify(localUiPreferences62));
  enqueueStorageWrite(APP_DB_KEY,db);
}
function applyUiPreferences62(navigate=false){
  ensureUiPreferenceOwner62();
  const prefs=localUiPreferences62,t=today();
  restoringUiPreferences62=true;
  try{
    period=prefs.lastPeriod;
    let from=period==='custom'?prefs.from:'',to=period==='custom'?prefs.to:'';
    if(period!=='all'&&period!=='custom'){
      to=t;from=t;
      if(period==='month')from=t.slice(0,7)+'-01';
      else if(period==='7d'){const d=new Date(t+'T12:00:00');d.setDate(d.getDate()-6);from=localISO(d)}
    }
    if($('filterStart'))$('filterStart').value=from;
    if($('filterEnd'))$('filterEnd').value=to;
    $$('[data-period]').forEach(b=>b.classList.toggle('active',b.dataset.period===period));
    if(navigate)goPage(prefs.lastPage);
  }finally{restoringUiPreferences62=false}
  scheduleTableEnhancements62();
}
```

Schema 12 aceita e normaliza versões antigas. Preferências de colunas/sort/página/período/from/to vão em db.uiPrefs, preservando metadados extras (incluindo backup). Cache auxiliar local usa user.id disponível ou “local”; normalize/merge/retorno da nuvem recebem as mesmas preferências locais, sem aguardar rede ou criar histórico de commits. A carga só inicializa o cache depois de loadPersistentState, protegendo backups que só tenham db.uiPrefs. Hoje/7d/mês recalculam datas ao reabrir; custom preserva datas fixas. migrateData também conserva stockAfter finito e mantém undefined quando o campo não existe nos legados.

Teste manual: Escolha ordenação/colunas, último período e página, recarregue. Avance o relógio um dia e confira Hoje no dia novo; custom deve manter as datas. Alterne usuários e confira que as preferências voltam por usuário. Sincronização deve manter os dados de negócio e as preferências locais, sem aceitar preferências remotas.



### 5.13 — até três toasts empilhados

Funções: toast, toastStack62/removeToast62/refreshToastUndoButtons62. O container original #toast recebe itens independentes, novos primeiro, até três; cada item sai após2800ms. Texto é inserido por textContent; listeners de botões pertencem ao item e são descartados quando o item sai.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function toast(msg){let el=$('toast');if(!el)return;el.textContent=msg;el.classList.add('show');clearTimeout(toastTimer);toastTimer=setTimeout(()=>el.classList.remove('show'),2800)}
+function toast(msg,options={}){toastStack62(msg,options)}
```

```diff
+function removeToast62(item){clearTimeout(item.timer);item.node.remove();const index=activeToasts62.indexOf(item);if(index>=0)activeToasts62.splice(index,1)}
+function refreshToastUndoButtons62(){for(const item of activeToasts62){if(!item.undoButton)continue;const current=undoStack[undoStack.length-1]===item.undoEntry;item.undoButton.disabled=!current;item.undoButton.title=current?'Desfazer esta alteração':'Desfaça primeiro as alterações mais recentes pelo botão Desfazer'}}
+function toastStack62(message,options={}){
+  const root=$('toast');if(!root)return;root.className='toast-stack';root.setAttribute('aria-atomic','false');
+  const node=document.createElement('div');node.className='toast-item';const label=document.createElement('span');label.textContent=String(message);node.append(label);
+  const item={node,undoEntry:options.undoEntry,timer:null,undoButton:null};
+  if(options.undoEntry){const button=document.createElement('button');button.type='button';button.className='toast-undo';button.textContent='Desfazer';button.addEventListener('click',()=>{if(undoStack[undoStack.length-1]!==item.undoEntry){refreshToastUndoButtons62();return}undoLast();removeToast62(item);refreshToastUndoButtons62()});node.append(button);item.undoButton=button}
+  activeToasts62.unshift(item);root.prepend(node);while(activeToasts62.length>3)removeToast62(activeToasts62[activeToasts62.length-1]);item.timer=setTimeout(()=>removeToast62(item),2800);refreshToastUndoButtons62();
+}
```

Teste manual: provocar quatro mensagens rápidas; conferir somente três, quarta no topo; esperar2,8s e confirmar remoção independente. Testar Desfazer sem mensagem posterior apagar a anterior.



### 5.14 — estado real da gravação e retry

Funções: enqueueStorageWrite/updateSaveStatus/updateLocalSaveStatus62/retryLocalSave62; storage.get/set/remove. Só gravações APP_DB_KEY contam no indicador: pending mostra spinner/Salvando…, conclusão efetiva mostra hora em verde, rejeição mostra Erro ao salvar/Tentar de novo. Retry captura o db atual, não um snapshot antigo. A cadeia continua tratada e persist/commit seguem síncronos, com clone na entrada.

O root reproduziu uma falha anterior: transação IDB rejeitada com conexão ainda funcional gravava fallback localStorage, mas get/reload restauravam IDB antigo. Foram adicionados marcadores por chave para selecionar o último backend canônico; fallback de exclusão usa tombstone, e IDB subsequente bem-sucedido limpa o marcador. Evidências: fallback-transaction-before.json (falha) e after/combined (aprovado). O status só declara sucesso quando a próxima leitura preserva a gravação.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1,2 @@
-function fallbackGet(key)
+function fallbackMarker(key){return 'receitaProStorageFallback:'+key}
+  function fallbackGet(key)
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-async get(key){try{return await idb('readonly',st=>st.get(key))}catch(e){return fallbackGet(key)}},
+async get(key){try{const marker=localStorage.getItem(fallbackMarker(key));if(marker==='deleted')return null;if(marker==='1')return fallbackGet(key)}catch(e){console.warn('Marcador de armazenamento local indisponível',e)}try{return await idb('readonly',st=>st.get(key))}catch(e){return fallbackGet(key)}},
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-async set(key,value){try{await idb('readwrite',st=>st.put(value,key))}catch(e){try{localStorage.setItem(key,JSON.stringify(value))}catch(err){throw err}}},
+async set(key,value){try{await idb('readwrite',st=>st.put(value,key))}catch(e){localStorage.setItem(key,JSON.stringify(value));localStorage.setItem(fallbackMarker(key),'1');return}try{localStorage.removeItem(fallbackMarker(key))}catch(e){console.warn('Não foi possível limpar marcador de fallback',e);localStorage.setItem(key,JSON.stringify(value));localStorage.setItem(fallbackMarker(key),'1')}},
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-async remove(key){try{await idb('readwrite',st=>st.delete(key))}catch(e){try{localStorage.removeItem(key)}catch(err){}}}
+async remove(key){try{await idb('readwrite',st=>st.delete(key))}catch(e){localStorage.removeItem(key);localStorage.setItem(fallbackMarker(key),'deleted');return}localStorage.removeItem(key);localStorage.removeItem(fallbackMarker(key))}
--- ANTES
+++ DEPOIS
@@ -1 +1,11 @@
-function enqueueStorageWrite(key,value){const snapshot=deepClone(value);storageWriteChain=storageWriteChain.then(()=>storage.set(key,snapshot)).catch(e=>{console.error('Falha ao persistir',key,e);Promise.resolve().then(()=>ui.alert('O navegador não conseguiu salvar os dados. Exporte um backup antes de fechar a página.','Falha ao salvar')).catch(err=>console.warn('Aviso de falha ao salvar',err))});return storageWriteChain}
+function enqueueStorageWrite(key,value){
+  const snapshot=deepClone(value),main=key===APP_DB_KEY,writeId=main?++localSaveState62.latestWrite:0;
+  if(main){localSaveState62.pending++;localSaveState62.error=null;updateSaveStatus()}
+  storageWriteChain=storageWriteChain.then(()=>storage.set(key,snapshot)).then(()=>{
+    if(main){localSaveState62.pending--;localSaveState62.savedAt=Date.now();if(writeId===localSaveState62.latestWrite)localSaveState62.error=null;updateSaveStatus()}
+  },e=>{
+    console.error('Falha ao persistir',key,e);
+    if(main){localSaveState62.pending--;if(writeId===localSaveState62.latestWrite){localSaveState62.error=e;toast('Não foi possível salvar. Tente novamente ou exporte um backup.')}updateSaveStatus()}
+    else toast('Não foi possível salvar um ponto de restauração. Exporte um backup.');
+  });return storageWriteChain;
+}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function updateSaveStatus(){let time=new Date().toLocaleTimeString('pt-BR',{hour:'2-digit',minute:'2-digit'});$('saveStatus').textContent=cloud.workspace?(cloud.dirty?'Salvo local • sincronizando':'Sincronizado'):`Salvo às ${time}`;$('sideMode').textContent=cloud.workspace?'Nuvem conectada':'Modo local';$('sideSave').textContent=cloud.workspace?(cloud.dirty?'Há alterações pendentes':'Dados sincronizados entre aparelhos'):'Seus dados ficam neste aparelho'}
+function updateSaveStatus(){updateLocalSaveStatus62()}
```

Teste manual: atrasar storage.set e observar Salvando…; rejeitar gravação e observar erro/retry; alterar estado e clicar Tentar de novo, confirmando snapshot recente. Simular falha de transação IDB com localStorage disponível, recarregar e confirmar o valor salvo, mesmo com IDB novamente funcional.



### 5.15 — alto contraste opcional aplicado

Funções: applyTheme/toggleTheme/initTheme e Ajustes>Acessibilidade>accessibilityTheme62. Os três temas preservam seleção local. Alto contraste define fundo#000/texto#fff/bordas2px; inclui FAB, nav badges, busy indicator, toasts, status e skeleton. Mantém a única definição global .sidebar e o único focus-visible do BlocoC; nav ativo usa marcador branco.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function applyTheme(theme){theme=theme==='dark'?'dark':'light';document.documentElement.dataset.theme=theme;safeLocalSet(THEME_KEY,theme);$('themeBtn').textContent=theme==='dark'?'☀':'☾';$('themeMeta')?.setAttribute('content',theme==='dark'?'#0b1210':'#0f3b2f');drawRevenueChart()}
+function applyTheme(theme){theme=['light','dark','high-contrast'].includes(theme)?theme:'light';document.documentElement.dataset.theme=theme;safeLocalSet(THEME_KEY,theme);const button=$('themeBtn'),labels={light:'claro',dark:'escuro','high-contrast':'alto contraste'},next={light:'dark',dark:'high-contrast','high-contrast':'light'};if(button){button.textContent=theme==='dark'?'◐':theme==='high-contrast'?'☀':'☾';const label=`Tema ${labels[theme]}. Alterar para ${labels[next[theme]]}.`;button.setAttribute('aria-label',label);button.title=label}$('themeMeta')?.setAttribute('content',theme==='high-contrast'?'#000000':theme==='dark'?'#0b1210':'#0f3b2f');const select=$('accessibilityTheme62');if(select)select.value=theme;drawRevenueChart()}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function toggleTheme(){applyTheme(document.documentElement.dataset.theme==='dark'?'light':'dark')}
+function toggleTheme(){const themes=['light','dark','high-contrast'],current=themes.indexOf(document.documentElement.dataset.theme);applyTheme(themes[(current+1)%themes.length])}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function initTheme(){let saved=localStorage.getItem(THEME_KEY);if(!saved)saved=window.matchMedia?.('(prefers-color-scheme: dark)').matches?'dark':'light';applyTheme(saved)}
+function initTheme(){let saved='';try{saved=localStorage.getItem(THEME_KEY)||''}catch(e){console.warn('Falha ao ler tema salvo',e)}if(!['light','dark','high-contrast'].includes(saved))saved=window.matchMedia?.('(prefers-color-scheme: dark)')?.matches?'dark':'light';applyTheme(saved)}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-dark=document.documentElement.dataset.theme==='dark',guide=dark?'#223129':'#e5eee8',label=dark?'#a8bbb2':'#52655c'
+theme=document.documentElement.dataset.theme,guide=theme==='high-contrast'?'#fff':theme==='dark'?'#223129':'#e5eee8',label=theme==='high-contrast'?'#fff':theme==='dark'?'#a8bbb2':'#52655c'
```

Trechos adicionais reais de alto contraste:

```css
html[data-theme="high-contrast"] :is(.btn,.nav button,.brand-mark,.metric-icon,.install-icon,.wallet-icon,.toast,.sync-mini,.side-btn,.topbar,.toast-item,.toast-undo,.save-chip,.fab,.nav-badge62,#cloudSkeleton62){background:#000;color:#fff;border:2px solid #fff;box-shadow:none}
html[data-theme="high-contrast"] body.busy:after{background:#000;color:#fff;border:2px solid #fff}
```

Teste manual: selecionar os três temas e recarregar; no alto contraste inspecionar FAB“+”, badges e mensagem Sincronizando: preto/branco/borda2px, contraste21:1. Foco continua único e visível.



### 5.16 — 50 registros por vez em tabelas grandes

Status: aplicado. Funções/arquivo: tableState62; enhanceTable62; setupTableControls62.

ANTES (trecho real):

```js
function renderTable(tbodyId,html){const tbody=$(tbodyId);if(!tbody)return;tbody.innerHTML=html;const table=tbody.closest('table[data-card-view="true"]');if(table)applyTableLabels(table)}
```

DEPOIS (trecho real):

```js
const paged=rows.length>100,limit=paged?state.limit:Infinity,shown=new Set(matching.slice(0,limit));
const more=document.createElement('button');more.type='button';more.className='btn ghost small';more.dataset.tableMore=id;more.textContent='Carregar mais';more.setAttribute('aria-label',`Carregar mais 50 registros de ${definition.name}`);more.addEventListener('click',()=>{tableState62(id).limit+=50;enhanceTables62()});footer.append(count,more);wrap.after(footer);
```

Acima de 100 linhas, o estado em memória começa em 50 e Carregar mais acrescenta 50. O limite permanece ao trocar de página. Busca/sort recomeçam no primeiro lote; o contador descreve o que está mostrado.

Teste manual: Com 125 compras, confira 50 visíveis, depois 100 e 125 via Carregar mais. Vá a Vendas e volte: o limite escolhido deve permanecer. Com 100 ou menos, todas as linhas devem aparecer.



### 5.17 — duplicar como variação

Funções: duplicateRecipe/renderRecipes. A cópia abre formulário novo sem ID/nome/preço, conserva ingredientes/quantidades/rendimento/custo extra; o botão ganhou o rótulo Duplicar como variação. Nenhum produto novo é criado antes de salvar.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function duplicateRecipe(id){let r=recipe(id);if(!r)return;goPage('recipes');resetRecipeForm();$('recipeItems').innerHTML='';$('recipeName').value=r.name+' — cópia';$('recipeYield').value=r.yield;$('recipePrice').value=r.price;$('recipeExtra').value=r.extra;for(const it of r.items||[])addRecipeItem(it.ingredientId,it.qty);previewRecipe();focusId('recipeName')}
+function duplicateRecipe(id){let r=recipe(id);if(!r)return;goPage('recipes');resetRecipeForm();$('recipeItems').innerHTML='';$('recipeName').value='';$('recipeYield').value=r.yield;$('recipePrice').value='';$('recipeExtra').value=r.extra;for(const it of r.items||[])addRecipeItem(it.ingredientId,it.qty);previewRecipe();focusId('recipeName')}
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-data-action="duplicateRecipe" data-id="${esc(r.id)}">Duplicar</button>
+data-action="duplicateRecipe" data-id="${esc(r.id)}">Duplicar como variação</button>
```

Teste manual: duplicar uma receita, conferir nome/preço vazios e custos/insumos preservados; preencher nome/preço e salvar como nova variação sem alterar o original.



### 5.18 — imprimir listas de compras

Funções: printShoppingList/printDayShoppingList/shoppingRowsForRecipe62/shoppingPrintTable62/showShoppingPrint62 e botões no planejador/plano do dia. Usam o printReport e media print puros existentes, com faltas, embalagens inteiras e custo; nomes são escapados. A impressão é somente leitura e o fechamento diário permanece disponível.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-<button class="btn ghost" onclick="copyShoppingList()" type="button">Copiar lista de compras</button>
+<button class="btn ghost" onclick="copyShoppingList()" type="button">Copiar lista de compras</button><button class="btn ghost" id="printShoppingListBtn62" type="button">Imprimir lista de compras</button>
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-<button class="btn soft" onclick="copyDayShoppingList()" type="button">Copiar lista de compras</button>
+<button class="btn soft" onclick="copyDayShoppingList()" type="button">Copiar lista de compras</button><button class="btn ghost" id="printDayShoppingListBtn62" type="button">Imprimir lista de compras</button>
```

```diff
+function showShoppingPrint62(body){
+  const report=$('printReport');if(!report)return toast('A impressão não está disponível neste navegador.');
+  report.innerHTML=`<h1>${esc(db.businessName||'ReceitaPro')} — lista de compras</h1><p>Gerado em ${new Date().toLocaleString('pt-BR')}.</p>${body}`;
+  window.print();
+}
+function printShoppingList(){
+  const r=recipe($('planRecipe')?.value);if(!r)return toast('Selecione um produto no planejador primeiro.');
+  const qty=Math.max(1,Math.round(num($('planQty')?.value))),rows=shoppingRowsForRecipe62(r,qty);
+  showShoppingPrint62(`<h2>Produzir ${qty} un. de ${esc(r.name)}</h2>${shoppingPrintTable62(rows)}`);
+}
+function printDayShoppingList(){
+  const plan=getDayPlan();if(!plan.units)return toast('Monte o plano do dia primeiro.');
+  const choices=`<ul>${plan.choices.map(x=>`<li>${esc(x.r.name)}: ${x.qty} un.</li>`).join('')}</ul>`;
+  showShoppingPrint62(`<h2>Plano do dia — ${plan.units} un.</h2>${choices}<h2>Lista de compras</h2>${shoppingPrintTable62(plan.shopping)}<p>Custo consumido: ${money(plan.cost)}<br>Faturamento potencial: ${money(plan.revenue)}<br>Lucro bruto potencial: ${money(plan.profit)}</p>`);
+}
```

Teste manual: calcular planejamento e imprimir lista; conferir quantidades faltantes/embalagens/custo e que somente relatório aparece; imprimir plano do dia com vários produtos; repetir fechamento diário e confirmar que continua correto.



### 5.19 — lembrete de backup

Funções: backupTimestamp62/backupMetadata62/saveBackupMetadata62/ensureBackupTracking62/renderBackupReminder62/recordBackupExport62/dismissBackupReminder62/exportData; banner no topo de Ajustes. Referência sem exportação é backupTrackingStartedAt, gravada uma vez; commits/updatedAt não adiam o aviso. A partir de sete dias, banner oferece exportar ou dispensar por mais sete.

ExportData registra timestamp somente depois de gerar/iniciar download; imprimir não registra exportação. Metadados por usuário usam saveUiPreferences62(patch); há fallback local seguro quando o helper não está disponível. Parser aceita epoch numérico, string numérica e ISO, rejeitando valores inválidos/fora do intervalo Date; datas futuras válidas são necessárias para o fim da dispensa.

```diff
--- ANTES
+++ DEPOIS
@@ -1 +1,2 @@
 <div class="page-head"><div><h2>Ajustes, segurança e nuvem</h2><p>Backup, integridade dos dados e sincronização entre aparelhos.</p></div></div>
+<div class="banner info hide" id="backupReminder62" role="status" aria-live="polite"><div class="bicon" aria-hidden="true">↥</div><div><strong id="backupReminderMessage62"></strong><div class="form-actions"><button class="btn primary small" id="backupExportNow62" type="button">Exportar agora</button><button class="btn ghost small" id="backupDismiss62" type="button">Dispensar por 7 dias</button></div></div></div>
--- ANTES
+++ DEPOIS
@@ -1 +1 @@
-function exportData(){try{downloadText(JSON.stringify(db,null,2),`backup-receitapro-${today()}.json`,'application/json');toast('Backup completo gerado.')}catch(e){console.error('exportData',e);ui.alert('Não foi possível gerar o backup. Tente novamente ou use outro navegador.')}}
+function exportData(){try{downloadText(JSON.stringify(db,null,2),`backup-receitapro-${today()}.json`,'application/json');recordBackupExport62();toast('Backup completo gerado.')}catch(e){console.error('exportData',e);ui.alert('Não foi possível gerar o backup. Tente novamente ou use outro navegador.')}}
```

```diff
+function backupTimestamp62(value){
+  const numeric=typeof value==='number'||(typeof value==='string'&&/^\d+$/.test(value));
+  const parsed=numeric?Number(value):typeof value==='string'?Date.parse(value):0;
+  return Number.isFinite(parsed)&&parsed>0&&parsed<=8640000000000000?parsed:0;
+}
```

Teste manual: setar lastBackupExportAt de8dias atrás, visitar Ajustes e conferir texto; dispensar e recarregar; avançar7dias e conferir retorno; exportar com sucesso esconde aviso e atualiza referência, falhar download mantém referência antiga. Verificar que impressão/commits não renovam o prazo.



### 5.20 — aplicado

Funções: initQuickAudit62, openQuickAudit62, renderQuickAudit62, stockAuditDifference62; listener adjustForm. AltC/botão em Contagem abrem saldo e variação por insumo; Contar chama startCount existente. Novo stockAfter conserva saldo observado para current-stockAfter, preservando precisão após editar/excluir movimentos antigos. Ajustes legados reconstroem baseline pelo histórico com aviso de estimativa; sem horário usam data. Delta/validação e nenhum-ajuste-quando-delta-zero permanecem.

Manual: inicial100+compra40,ajuste−10,consumo20,compra7→saldo117/variação−13. Contar110→ajuste−7/stockAfter110/variação0; recarregar e conferir. Legados indicam estimativa; Contar focaliza quantidade na tela original.

ANTES (trecho real):

```javascript
$('adjustForm').addEventListener('submit',e=>{e.preventDefault();let i=ingredient($('adjustIngredient').value),counted=num($('adjustCounted').value),date=$('adjustDate').value,reason=$('adjustReason').value;if(!i||counted<0||!date)return ui.alert('Confira os dados da contagem.');let current=ingredientStock(i.id),delta=counted-current;if(Math.abs(delta)<.000001)return toast('O saldo já está igual à contagem. Nenhum ajuste necessário.');commit('Estoque ajustado pela contagem',()=>db.stockAdjustments.push({id:uid(),date,ingredientId:i.id,delta,reason,createdAt:new Date().toISOString()}));$('adjustCounted').value='';updateAdjustPreview()});
```

DEPOIS (trecho real):

```javascript
$('adjustForm').addEventListener('submit',e=>{e.preventDefault();let i=ingredient($('adjustIngredient').value),counted=num($('adjustCounted').value),date=$('adjustDate').value,reason=$('adjustReason').value;if(!i||counted<0||!date)return ui.alert('Confira os dados da contagem.');let current=ingredientStock(i.id),delta=counted-current;if(Math.abs(delta)<.000001)return toast('O saldo já está igual à contagem. Nenhum ajuste necessário.');commit('Estoque ajustado pela contagem',()=>db.stockAdjustments.push({id:uid(),date,ingredientId:i.id,delta,reason,stockAfter:counted,createdAt:new Date().toISOString()}));$('adjustCounted').value='';updateAdjustPreview()});
function stockAuditDifference62(item){
  const adjustments=db.stockAdjustments.filter(adjustment=>adjustment.ingredientId===item.id);
  const timestamp=movement=>{
    const registered=Date.parse(movement.createdAt||'');
    return Number.isFinite(registered)?registered:Date.parse((movement.date||'1970-01-01')+'T23:59:59');
  };
  const last=adjustments.reduce((latest,adjustment)=>!latest||timestamp(adjustment)>=timestamp(latest)?adjustment:latest,null);
  const current=ingredientStock(item.id);
  if(!last)return {current,difference:current-num(item.initialStock),reference:'Desde o saldo inicial',estimated:true};
  if(last.stockAfter!==undefined&&last.stockAfter!==null&&Number.isFinite(Number(last.stockAfter)))return {current,difference:current-num(last.stockAfter),reference:`Último ajuste: ${formatDate(last.date)} • saldo contado ${qtyText(last.stockAfter,item.unit)}`,estimated:false};
  let estimated=!Number.isFinite(Date.parse(last.createdAt||''));
  const before=movement=>{
    const moveTime=Date.parse(movement.createdAt||''),lastTime=Date.parse(last.createdAt||'');
    if(Number.isFinite(moveTime)&&Number.isFinite(lastTime))return moveTime<=lastTime;
    estimated=true;return (movement.date||'')<=(last.date||'');
  };
  const bought=db.purchases.filter(purchase=>purchase.ingredientId===item.id&&before(purchase)).reduce((sum,purchase)=>sum+num(purchase.qty),0);
  const adjusted=adjustments.filter(before).reduce((sum,adjustment)=>sum+num(adjustment.delta),0);
  const consumed=db.production.filter(production=>(production.consumedItems||[]).some(consumption=>consumption.ingredientId===item.id)&&before(production)).reduce((sum,production)=>sum+(production.consumedItems||[]).filter(consumption=>consumption.ingredientId===item.id).reduce((subtotal,consumption)=>subtotal+num(consumption.qty),0),0);
  const baseline=num(item.initialStock)+bought+adjusted-consumed;
  return {current,difference:current-baseline,reference:`Último ajuste: ${formatDate(last.date)} • estimativa pelo histórico${estimated?' sem horário':''}`,estimated:true};
}
```



## Complementos de integração e auditoria final

### Conexão de workspace preserva alterações feitas durante a espera

Função: `cloudOpenWorkspace`. Antes, o snapshot local era capturado antes do await do RPC; uma operação local nesse intervalo podia desaparecer. Agora o snapshot é capturado depois da resposta, antes da decisão de merge. Teste: atrasar gp_get_data, cadastrar um insumo e concluir conexão; o insumo local e o remoto permanecem em memória e no IndexedDB.


ANTES:

```js
async function cloudOpenWorkspace(w){const base=await loadSyncBase(w.id),local=deepClone(normalizedData(db)),localHas=hasBusinessData(local),state=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:w.id}),
```

DEPOIS:

```js
async function cloudOpenWorkspace(w){const base=await loadSyncBase(w.id),state=await cloudRequest('/rest/v1/rpc/gp_get_data','POST',{p_workspace_id:w.id}),local=deepClone(normalizedData(db)),localHas=hasBusinessData(local),
```


### Logs nos fallbacks e exposição consistente

Funções: `saveAutoBackup`, `queueDialog`, callback de onboarding, catch de inicialização, `copyShoppingList`, `copyDayShoppingList`, `copyInvite`. Os fallbacks existentes foram mantidos; recebem console.warn em vez de suprimir o diagnóstico. O objeto `exposed` também é acessível como `window.exposed`, com helpers/wrappers atualizados.

ANTES → DEPOIS (trecho real dos dois copiadores):

```diff
-.catch(()=>ui.alert(text))
+.catch(e=>{console.warn('Não foi possível copiar a lista',e);return ui.alert(text)})
```

ANTES → DEPOIS (trecho real de convite):

```diff
-.catch(()=>toast('Selecione e copie o código manualmente.'))
+.catch(e=>{console.warn('Não foi possível copiar o convite',e);toast('Selecione e copie o código manualmente.')})
```

Manual: negar acesso ao clipboard; a lista continua disponível por alerta e o convite orienta cópia manual, com aviso no console. Teste automatizado: três rejeições controladas produziram três warnings, dois alertas e toast, zero exceções/rejeições órfãs.


## Identificação do arquivo final

SHA-256 de `ReceitaPro_6.2_FINAL.html`:

```text
b8e62cc47c8672e125907a4e5fc5978a1cd926a83625d23364cf532b26e16b0a
```

Evidências preservadas em `/workspace/scratch/receitapro-tests`: `stage1-final.json`, `stage5-integration-final.json`, `compat-final.json`, `cloud-final.json`, `cloud-open-race-final.json`, `fallback-transaction-final.json`, `clipboard-final.json`, `stage3-final.json`, `stage4-final.json` e `visual-final.json`.
