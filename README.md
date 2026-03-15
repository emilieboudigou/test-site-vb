<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>VétRôle — Simulation Communication Vétérinaire</title>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{font-family:'Segoe UI',Roboto,sans-serif;background:#1a1a2e;overflow:hidden;width:100vw;height:100vh}
canvas{display:block}
#ui{position:absolute;top:0;left:0;width:100%;height:100%;pointer-events:none}

#hud{position:absolute;top:12px;left:50%;transform:translateX(-50%);background:rgba(15,30,60,0.85);backdrop-filter:blur(12px);border:1px solid rgba(78,205,196,0.3);border-radius:30px;padding:8px 24px;color:white;font-size:13px;display:flex;gap:18px;align-items:center;pointer-events:none}
#score-hud{color:#ffd700;font-weight:800;font-size:14px}
#phase-hud{color:#a8e6cf}

#interact-prompt{position:absolute;bottom:70px;left:50%;transform:translateX(-50%);background:rgba(10,20,40,0.9);backdrop-filter:blur(8px);border:2px solid #4ecdc4;border-radius:16px;padding:10px 24px;color:white;font-size:14px;display:none;text-align:center;pointer-events:none;box-shadow:0 4px 20px rgba(78,205,196,0.3)}

/* TITRE */
#title-screen{position:absolute;inset:0;background:linear-gradient(160deg,#0d1b3e 0%,#1a3a5c 50%,#0d1b3e 100%);display:flex;flex-direction:column;align-items:center;justify-content:center;pointer-events:all;overflow:hidden}
.title-bg-paw{position:absolute;font-size:200px;opacity:0.04;user-select:none}
.title-bg-paw:nth-child(1){top:-40px;left:-60px;transform:rotate(-20deg)}
.title-bg-paw:nth-child(2){bottom:-60px;right:-40px;transform:rotate(15deg)}
#title-screen h1{font-size:58px;color:white;font-weight:900;letter-spacing:-1px;text-shadow:0 0 40px rgba(78,205,196,0.6);margin-bottom:4px;position:relative}
.title-vet{color:#4ecdc4}
.title-role{color:white}
#title-screen .tagline{font-size:14px;color:rgba(168,230,207,0.8);letter-spacing:3px;text-transform:uppercase;margin-bottom:8px}
#title-screen .desc{font-size:15px;color:rgba(255,255,255,0.55);max-width:520px;text-align:center;line-height:1.7;margin-bottom:10px}
.level-badge{background:linear-gradient(135deg,#e74c3c,#c0392b);color:white;font-size:12px;font-weight:700;padding:5px 16px;border-radius:20px;letter-spacing:1px;margin-bottom:30px}
.btn-start{background:linear-gradient(135deg,#4ecdc4,#26a69a);border:none;color:white;font-size:17px;font-weight:700;padding:16px 52px;border-radius:50px;cursor:pointer;transition:all 0.3s;box-shadow:0 8px 32px rgba(78,205,196,0.35);pointer-events:all;letter-spacing:0.5px}
.btn-start:hover{transform:translateY(-3px);box-shadow:0 14px 40px rgba(78,205,196,0.55)}
.title-profiles{display:flex;gap:14px;margin-top:28px;flex-wrap:wrap;justify-content:center}
.profile-pill{background:rgba(255,255,255,0.07);border:1px solid rgba(255,255,255,0.15);border-radius:20px;padding:6px 16px;color:rgba(255,255,255,0.7);font-size:13px}
.intro-anim{animation:floatIn 0.8s ease both}
@keyframes floatIn{from{opacity:0;transform:translateY(20px)}to{opacity:1;transform:translateY(0)}}

/* CONSULTATION */
#consult-overlay{position:absolute;inset:0;background:rgba(5,10,25,0.7);backdrop-filter:blur(6px);display:none;pointer-events:all;align-items:flex-end;justify-content:center;padding-bottom:0}
#consult-box{background:#f9f9fb;border-radius:28px 28px 0 0;width:100%;max-width:860px;max-height:88vh;display:flex;flex-direction:column;box-shadow:0 -24px 80px rgba(0,0,0,0.6);animation:slideUp 0.4s cubic-bezier(0.34,1.56,0.64,1)}
@keyframes slideUp{from{transform:translateY(100%);opacity:0}to{transform:translateY(0);opacity:1}}
.consult-header{padding:20px 24px 16px;border-bottom:1px solid #ebebeb;display:flex;align-items:center;gap:14px;flex-shrink:0;background:white;border-radius:28px 28px 0 0}
.owner-avatar-big{width:64px;height:64px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:34px;flex-shrink:0;box-shadow:0 4px 12px rgba(0,0,0,0.1)}
.consult-header h2{font-size:18px;color:#1a1a2e;font-weight:800;margin-bottom:2px}
.consult-header .animal-line{font-size:13px;color:#7f8c8d}
.personality-badge{display:inline-block;padding:3px 12px;border-radius:20px;font-size:11px;font-weight:700;margin-top:4px}
.close-btn{margin-left:auto;background:#fee2e2;border:none;border-radius:14px;padding:9px 18px;cursor:pointer;font-weight:700;color:#dc2626;pointer-events:all;font-size:13px;transition:background 0.2s;flex-shrink:0}
.close-btn:hover{background:#fecaca}

/* Progress bar consultation */
.progress-bar-wrap{padding:0 24px;height:6px;background:#f0f0f0;position:relative;flex-shrink:0}
.progress-bar-fill{height:6px;background:linear-gradient(90deg,#4ecdc4,#26a69a);border-radius:3px;transition:width 0.5s ease}

.dialogue-area{flex:1;overflow-y:auto;padding:20px 24px;display:flex;flex-direction:column;gap:10px}
.bubble{padding:13px 17px;border-radius:18px;max-width:82%;font-size:14px;line-height:1.65;animation:fadeMsg 0.3s ease;flex-shrink:0}
@keyframes fadeMsg{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:translateY(0)}}
.bubble-owner{background:white;color:#2c3e50;margin-right:auto;border:1.5px solid #e8e8e8;border-bottom-left-radius:4px;box-shadow:0 2px 8px rgba(0,0,0,0.06)}
.bubble-owner::before{content:attr(data-name);display:block;font-size:11px;font-weight:700;color:#94a3b8;margin-bottom:5px;text-transform:uppercase;letter-spacing:0.5px}
.bubble-vet{background:linear-gradient(135deg,#4ecdc4,#26a69a);color:white;margin-left:auto;border-bottom-right-radius:4px;box-shadow:0 3px 12px rgba(78,205,196,0.3)}
.bubble-vet::before{content:"🩺 Vous (vétérinaire)";display:block;font-size:11px;font-weight:700;color:rgba(255,255,255,0.65);margin-bottom:5px;text-transform:uppercase;letter-spacing:0.5px}
.bubble-system{background:linear-gradient(135deg,#fef9c3,#fef08a);color:#713f12;text-align:center;margin:4px auto;font-size:13px;border-radius:12px;font-style:italic;max-width:100%;padding:10px 16px}

.feedback-box{border-radius:14px;padding:16px;animation:fadeMsg 0.35s ease;flex-shrink:0;border-left:4px solid}
.feedback-good{background:#f0fdf4;border-color:#22c55e}
.feedback-warn{background:#fffbeb;border-color:#f59e0b}
.feedback-bad{background:#fef2f2;border-color:#ef4444}
.feedback-box .fb-score{font-size:22px;font-weight:900;margin-bottom:4px}
.feedback-box h4{font-size:13px;font-weight:700;margin-bottom:6px;color:#1a1a2e}
.feedback-box p{font-size:13px;line-height:1.6;color:#4a5568}
.best-answer{margin-top:10px;padding:10px;background:rgba(34,197,94,0.1);border-radius:8px;font-size:13px;color:#15803d;font-style:italic}

.options-area{padding:14px 24px 20px;display:flex;flex-direction:column;gap:8px;flex-shrink:0;background:white;border-top:1px solid #f0f0f0}
.opt-btn{background:#f8fafc;border:2px solid #e2e8f0;border-radius:14px;padding:13px 17px;text-align:left;cursor:pointer;font-size:13.5px;color:#1e293b;transition:all 0.22s;pointer-events:all;line-height:1.5;display:flex;align-items:flex-start;gap:10px}
.opt-btn:hover:not(:disabled){border-color:#4ecdc4;background:#f0fffc;transform:translateX(5px);box-shadow:0 3px 12px rgba(78,205,196,0.15)}
.opt-btn:disabled{opacity:0.45;cursor:not-allowed;transform:none}
.opt-key{background:#e2e8f0;color:#64748b;font-size:11px;font-weight:800;padding:2px 8px;border-radius:6px;flex-shrink:0;margin-top:2px}

/* RÉSULTATS */
#result-screen{position:absolute;inset:0;background:rgba(5,10,25,0.92);display:none;align-items:center;justify-content:center;pointer-events:all;padding:20px;overflow-y:auto}
.result-card{background:white;border-radius:28px;padding:36px;max-width:680px;width:100%;text-align:center;animation:floatIn 0.6s ease}
.result-card h2{font-size:26px;font-weight:900;color:#1a1a2e}
.result-emoji{font-size:64px;margin:14px 0 10px}
.score-ring{width:110px;height:110px;border-radius:50%;background:conic-gradient(#4ecdc4 var(--pct), #e2e8f0 0);display:flex;align-items:center;justify-content:center;margin:16px auto;box-shadow:0 8px 24px rgba(78,205,196,0.3);position:relative}
.score-ring-inner{width:82px;height:82px;border-radius:50%;background:white;display:flex;align-items:center;justify-content:center;font-size:26px;font-weight:900;color:#1a1a2e}
.recap-grid{display:grid;grid-template-columns:1fr 1fr;gap:10px;margin:18px 0;text-align:left}
.recap-item{background:#f8fafc;border-radius:14px;padding:14px;border-left:4px solid}
.recap-item h4{font-size:12px;font-weight:700;color:#94a3b8;text-transform:uppercase;letter-spacing:0.5px;margin-bottom:4px}
.recap-item p{font-size:14px;color:#1a1a2e;font-weight:600;line-height:1.5}
.pedagogie-box{background:linear-gradient(135deg,#f0fdf4,#dcfce7);border-radius:14px;padding:16px;margin:14px 0;text-align:left}
.pedagogie-box h4{font-size:13px;font-weight:800;color:#15803d;margin-bottom:8px}
.pedagogie-box li{font-size:13px;color:#374151;margin:5px 0;line-height:1.5;list-style:none;padding-left:4px}
.patients-done-bar{position:absolute;top:60px;right:15px;display:flex;flex-direction:column;gap:6px;pointer-events:none}
.patient-chip{background:rgba(15,30,60,0.85);border:1px solid rgba(78,205,196,0.3);border-radius:20px;padding:5px 14px;color:white;font-size:12px;display:flex;align-items:center;gap:7px;backdrop-filter:blur(8px)}

#controls-hint{position:absolute;bottom:14px;left:50%;transform:translateX(-50%);display:flex;gap:8px;pointer-events:none}
.key-hint{background:rgba(15,30,60,0.8);border:1px solid rgba(78,205,196,0.25);border-radius:8px;padding:5px 13px;color:rgba(255,255,255,0.7);font-size:11px}
</style>
</head>
<body>
<canvas id="c"></canvas>
<div id="ui">

  <!-- Écran titre -->
  <div id="title-screen">
    <div class="title-bg-paw">🐾</div>
    <div class="title-bg-paw">🐾</div>
    <div class="intro-anim" style="text-align:center">
      <p class="tagline" style="animation-delay:0.1s">Serious Game · Formation Vétérinaire</p>
      <h1><span class="title-vet">Vét</span><span class="title-role">Rôle</span></h1>
      <div class="level-badge" style="margin:12px auto 0;display:inline-block">🎓 NIVEAU AVANCÉ — 5ème année</div>
      <p class="desc" style="margin-top:16px">Affrontez des propriétaires aux comportements difficiles et apprenez à maîtriser l'art de la communication clinique. Chaque mot compte.</p>
      <button class="btn-start" onclick="startGame()">▶ &nbsp;Démarrer la simulation</button>
      <div class="title-profiles">
        <span class="profile-pill">😰 Paniquard</span>
        <span class="profile-pill">🧐 Le Sachant</span>
        <span class="profile-pill">😠 Agressif</span>
        <span class="profile-pill">😶 Dans le déni</span>
      </div>
    </div>
  </div>

  <!-- HUD -->
  <div id="hud" style="display:none">
    <span>🏥 Salle d'attente</span>
    <span id="phase-hud">📋 Choisissez un patient</span>
    <span id="score-hud">⭐ 0 pts</span>
    <span id="patients-count" style="color:#94a3b8">0 / 4 consultés</span>
  </div>

  <div class="patients-done-bar" id="patients-done"></div>

  <div id="interact-prompt"></div>

  <!-- Overlay consultation -->
  <div id="consult-overlay">
    <div id="consult-box">
      <div class="consult-header">
        <div class="owner-avatar-big" id="c-avatar"></div>
        <div>
          <h2 id="c-name"></h2>
          <div class="animal-line" id="c-animal"></div>
          <span class="personality-badge" id="c-badge"></span>
        </div>
        <button class="close-btn" onclick="closeConsult()">✕ Fin</button>
      </div>
      <div class="progress-bar-wrap"><div class="progress-bar-fill" id="progress-bar" style="width:0%"></div></div>
      <div class="dialogue-area" id="dialogue-area"></div>
      <div class="options-area" id="options-area"></div>
    </div>
  </div>

  <!-- Résultats -->
  <div id="result-screen">
    <div class="result-card">
      <div class="result-emoji" id="res-emoji"></div>
      <h2 id="res-title"></h2>
      <p id="res-sub" style="color:#64748b;font-size:14px;margin-top:6px"></p>
      <div class="score-ring" id="score-ring"><div class="score-ring-inner" id="res-score"></div></div>
      <p style="font-size:13px;color:#94a3b8;margin-bottom:16px">Score de communication</p>
      <div class="recap-grid" id="recap-grid"></div>
      <div class="pedagogie-box">
        <h4>📚 Synthèse pédagogique</h4>
        <ul id="res-tips"></ul>
      </div>
      <button class="btn-start" onclick="restartGame()" style="margin-top:6px;font-size:15px;padding:13px 38px">🔄 Nouvelle simulation</button>
    </div>
  </div>

  <div id="controls-hint" style="display:none">
    <div class="key-hint">Z Q S D · Déplacer</div>
    <div class="key-hint">E · Consulter</div>
    <div class="key-hint">Échap · Fermer</div>
  </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ═══════════════════════════════════════════════════════════
//  BASE DE CAS CLINIQUES — NIVEAU AVANCÉ (5e année)
//  Dialogues réalistes, feedback pédagogique détaillé
// ═══════════════════════════════════════════════════════════
const OWNERS = [
  // ────────────────────────────────────────────────────────
  //  CAS 1 : LA PANIQUARDE
  // ────────────────────────────────────────────────────────
  {
    id:0, name:"Mme Fontaine", animal:"Cannelle (chatte, 7 ans)", emoji:"😰",
    pet:"chat", petEmoji:"🐱",
    color:"#e74c3c", badgeColor:"#fde8e8", badgeText:"😰 Hyperanxieuse", badgeTC:"#b91c1c",
    intro:"Docteur, je suis dans un état... Cannelle a vomi deux fois ce matin et elle ne veut pas manger. J'ai passé la nuit à surveiller sa respiration. J'ai cherché sur Internet et j'ai vu qu'un chat qui ne mange pas peut développer une lipidose hépatique en 24 heures. C'est vrai ça ? Elle va mourir si on n'agit pas maintenant ?!",
    type:"panicky",
    steps:[
      {
        ownerMsg:"J'ai même une liste des symptômes que j'ai notés cette nuit. Regardez : deux vomissements à 3h07 et 4h22, pupilles normales à 3h15, respiration à 28 mouvements par minute à 5h... C'est trop élevé non ? Et ses gencives, elles me semblent un peu pâles, non ?",
        options:[
          {text:"Wow, 28 mouvements par minute c'est en effet dans la limite haute, ça peut effectivement être inquiétant... On va vite l'ausculter.", score:-15, quality:"bad",
           reason:"Vous amplifiez l'anxiété en validant une valeur normale (16-40 mvt/min chez le chat) comme suspecte. Face à un profil hyperanxieux, chaque signal ambigu que vous confirmez déclenche une spirale de panique. C'est une erreur classique de jeunes cliniciens qui cherchent à 'se montrer à l'écoute'."},
          {text:"Je comprends que vous ayez passé une nuit difficile — cette vigilance montre à quel point vous tenez à Cannelle. Cette liste va vraiment m'aider. Dites-moi : elle a accès à des plantes, des médicaments ou quelque chose d'inhabituel à la maison ?", score:20, quality:"good",
           reason:"Excellent. Vous validez l'émotion sans valider les interprétations anxieuses, vous valorisez l'outil concret (la liste), et vous redirigez immédiatement vers une anamnèse productive. C'est la technique du 'pivot empathique' : reconnaître → valoriser → recadrer."},
          {text:"Mme Fontaine, tout d'abord respirez. Internet est souvent alarmiste. Deux vomissements chez un chat, ça arrive très couramment, ne vous inquiétez pas.", score:-5, quality:"warn",
           reason:"Demander de 'respirer' et minimiser les symptômes d'emblée (sans examen) peut paraître condescendant. Le propriétaire se sentira incompris. La minimisation prématurée brise la confiance avant même le début de la consultation."},
          {text:"Votre suivi nocturne est très détaillé. Deux vomissements isolés avec un intervalle de plus d'une heure, c'est un élément que j'intègre. Je vais examiner Cannelle maintenant et on fera le point ensemble sur chaque paramètre que vous avez noté.", score:17, quality:"good",
           reason:"Bonne réponse : vous légitimisez le suivi sans dramatiser, et vous montrez que les données du propriétaire ont une valeur clinique. Vous positionnez l'examen comme une démarche structurée et rassurante."}
        ]
      },
      {
        ownerMsg:"Et la lipidose ? C'est vrai qu'elle peut mourir en 24 heures ? Elle n'a pas mangé ce matin... J'aurais dû venir hier soir ! C'est de ma faute !",
        options:[
          {text:"Non non, la lipidose hépatique se développe en général sur plusieurs jours de jeûne chez un chat obèse, pas en 24h. Cannelle n'a pas ce profil d'après ce que je vois.", score:15, quality:"good",
           reason:"Information médicale exacte et rassurante, bien délivrée. Vous corrigez une information erronée trouvée sur Internet avec précision. Petite réserve : il manque encore la gestion du sentiment de culpabilité exprimé."},
          {text:"Vous n'avez rien à vous reprocher. Et concernant la lipidose, je dois rester honnête avec vous : c'est possible, mais c'est une évolution sur plusieurs jours chez un chat obèse en jeûne prolongé. Cannelle ne présente pas encore ce tableau. L'important c'est qu'elle est là maintenant.", score:20, quality:"good",
           reason:"Réponse complète : vous absolvez la culpabilité (outil essentiel avec les profils anxieux), vous corrigez médicalement l'information, vous contextualisez, et vous ancrez dans le présent avec une note positive. Technique des 4 composantes."},
          {text:"Ne vous culpabilisez surtout pas, ça n'aide ni vous ni Cannelle. Parlons plutôt de ce qu'on va faire maintenant.", score:5, quality:"warn",
           reason:"La culpabilité est évacuée trop rapidement sans être réellement traitée. 'Ça n'aide pas' peut sonner comme un reproche. Mieux vaut valider l'émotion avant de la dépasser."},
          {text:"La lipidose ça se traite très bien aujourd'hui, ne vous inquiétez pas pour ça.", score:-8, quality:"bad",
           reason:"Vous répondez à côté : le propriétaire exprime de la culpabilité, pas seulement une peur médicale. Ignorer la dimension émotionnelle d'un message est une erreur de communication fréquente. Vous écoutez les mots mais pas le sous-texte."}
        ]
      },
      {
        ownerMsg:"D'accord... Qu'est-ce que vous avez trouvé à l'examen ? Dites-moi tout, je préfère savoir même si c'est grave. Mais en même temps... j'ai peur d'entendre quelque chose de terrible.",
        options:[
          {text:"Je vais vous dire exactement ce que j'ai trouvé. L'état général de Cannelle est bon : muqueuses roses, hydratation correcte, abdomen souple. Les deux vomissements peuvent avoir plusieurs origines bénignes. On va faire quelques examens ciblés. Je vous accompagne à chaque étape.", score:20, quality:"good",
           reason:"Réponse modèle : vous respectez son souhait d'information complète, vous annoncez les bonnes nouvelles d'abord (technique de la 'bonne nouvelle en premier'), vous proposez une suite structurée, et vous vous positionnez comme un allié. C'est la posture idéale avec un profil anxieux."},
          {text:"Alors, il faut que je vous dise... j'ai quelques inquiétudes sur le foie, on va avoir besoin de faire une prise de sang en urgence.", score:-20, quality:"bad",
           reason:"Énoncer des inquiétudes sans éléments cliniques précis face à un profil hyperanxieux est une faute grave. Vous allez déclencher une crise de panique pour rien. Règle d'or : jamais d'hypothèses alarmantes sans données cliniques solides."},
          {text:"Tout va bien, ne vous inquiétez pas. C'est probablement un trouble digestif bénin.", score:-8, quality:"bad",
           reason:"La banalisation totale après une nuit d'angoisse du propriétaire est perçue comme du mépris. De plus, vous concluez sans avoir présenté les résultats de l'examen, ce qui manque de rigueur professionnelle."},
          {text:"À l'examen, je retrouve un chat alerte, avec de bonnes muqueuses. Je vais quand même proposer une prise de sang pour avoir des données objectives — ça nous permettra de vous répondre avec certitude et ça écartera définitivement les hypothèses sérieuses.", score:16, quality:"good",
           reason:"Bonne approche : vous rassurez sur le clinique et proposez l'examen complémentaire comme outil de certitude plutôt que d'inquiétude. La nuance 'écartera définitivement' est une formulation rassurante très efficace."}
        ]
      }
    ],
    finalTip:"Profil PANIQUARD : L'outil clé est le 'pivot empathique' (valider l'émotion → valoriser la vigilance → recadrer vers les faits). Ne jamais minimiser prématurément. Ne jamais amplifier. Votre calme clinique est en soi thérapeutique."
  },

  // ────────────────────────────────────────────────────────
  //  CAS 2 : LE SACHANT
  // ────────────────────────────────────────────────────────
  {
    id:1, name:"M. Leclerc", animal:"Thor (Labrador, 4 ans)", emoji:"🧐",
    pet:"chien", petEmoji:"🐕",
    color:"#d97706", badgeColor:"#fef3c7", badgeText:"🧐 Le Sachant", badgeTC:"#92400e",
    intro:"Bonjour. Je vais aller droit au but : Thor a une dermatite atopique avec probable composante alimentaire. J'ai lu les dernières méta-analyses sur PubMed, j'ai rejoint un groupe Facebook de propriétaires de Labs avec vétérinaire consultant, et j'ai déjà commandé une alimentation hydrolysée. J'ai besoin que vous me prescriviez une corticothérapie à faible dose et que vous confirmiez mon diagnostic.",
    type:"knowitall",
    steps:[
      {
        ownerMsg:"J'ai imprimé trois études. Regardez — celle de 2022 montre que 68% des Labs présentant ce tableau ont une sensibilisation aux protéines bovines. Thor mange du bœuf depuis 3 ans. C'est du bon sens, non ? Vous ne pouvez pas nier les données.",
        options:[
          {text:"Vous avez tort sur l'interprétation. Ces études portent sur des populations, pas sur des individus. Thor n'est peut-être pas dans ce 68%.", score:-8, quality:"bad",
           reason:"Confrontation directe avec un 'sachant' : il va se braquer immédiatement. Même si votre raisonnement est juste, la forme est contre-productive. Avec ce profil, la confrontation frontale renforce ses positions plutôt qu'elle ne les change."},
          {text:"Vous avez fait un travail de documentation sérieux, je le reconnais. Ces données sont réelles. Le problème, c'est qu'une corrélation de population ne permet pas le diagnostic individuel — c'est exactement pourquoi j'ai besoin d'examiner Thor et de faire un bilan allergologique avant de prescrire une corticothérapie qui pourrait masquer d'autres pathologies.", score:20, quality:"good",
           reason:"Excellent. Vous utilisez la technique du 'oui… et' : vous validez ses données sans les rejeter, vous montrez votre propre maîtrise scientifique (corrélation ≠ causalité individuelle), et vous expliquez la logique médicale derrière votre démarche. Le 'sachant' répond bien au respect de son niveau."},
          {text:"Je comprends votre approche. Dites-moi, vous avez fait un régime d'éviction avant de consulter ?", score:12, quality:"warn",
           reason:"Question clinique pertinente qui montre votre compétence, mais vous esquivez la discussion sur ses études. Avec un 'sachant', il vaut mieux engager directement sur le fond — esquiver peut être perçu comme un aveu que vous n'avez pas lu les données."},
          {text:"Très bien documenté. La dermatite atopique est en effet une priorité diagnostique chez le Lab. Mais avant toute corticothérapie, j'ai besoin d'exclure une dermatite parasitaire, une pyodermite secondaire, et de valider le tableau clinique.", score:16, quality:"good",
           reason:"Bonne réponse professionnelle. Vous valorisez son travail et vous posez les jalons de votre démarche. La liste des diagnostics différentiels montre votre expertise et crédibilise votre approche."}
        ]
      },
      {
        ownerMsg:"Mais la corticothérapie courte durée n'a pas d'effets secondaires significatifs selon la littérature. Vous ne pouvez pas refuser de la prescrire juste pour respecter un protocole bureaucratique.",
        options:[
          {text:"Ce n'est pas bureaucratique, c'est déontologique. Je ne peux pas prescrire un traitement sans diagnostic établi.", score:5, quality:"warn",
           reason:"Vrai mais défensif. Se réfugier derrière la déontologie sans expliquer le pourquoi médical semble effectivement 'bureaucratique' à quelqu'un qui cite PubMed."},
          {text:"Vous avez raison que les corticoïdes courts sont généralement bien tolérés. Mais si Thor a une infection bactérienne sous-jacente — ce que je dois exclure à l'examen — les corticoïdes vont l'aggraver significativement. Je me retrouverais à vous avoir nui. Ce n'est pas un protocole, c'est de la médecine fondée sur les risques individuels.", score:20, quality:"good",
           reason:"Parfait. Vous utilisez la 'langue du sachant' (risques individuels, raisonnement clinique), vous donnez une raison médicale précise et convaincante (surinfection + corticoïdes = aggravation), et vous repositionnez comme quelqu'un qui refuse de nuire au patient. Inarguable."},
          {text:"Je suis le vétérinaire ici, et c'est moi qui décide des prescriptions.", score:-20, quality:"bad",
           reason:"Réponse d'autorité pure : catastrophique avec un 'sachant'. Il va se sentir méprisé, écrire un avis négatif, et peut-être partir voir un autre vétérinaire. Le rapport de force ne fonctionne jamais avec ce profil."},
          {text:"Montrez-moi exactement quelle étude parle de corticothérapie courte durée en première intention, je veux voir le protocole précis.", score:8, quality:"warn",
           reason:"Demander la source peut être perçu comme un défi ou une vérification. Si l'étude est solide, vous perdez du terrain. Mieux vaut proposer une alternative clinique convaincante."}
        ]
      },
      {
        ownerMsg:"...D'accord, je vous écoute. Qu'est-ce que vous proposez concrètement ? Parce que Thor souffre là, maintenant.",
        options:[
          {text:"Je propose : examen dermatologique complet maintenant, raclage cutané pour exclure la gale et la démodécie, cytologie pour les infections. Si tout est négatif, on débute le régime d'éviction que vous avez déjà commandé — votre initiative est donc utile. En 8 semaines on aura une réponse. Si la démangeaison est intense d'ici là, on peut utiliser de l'oclacitinib en attendant — plus ciblé que les corticoïdes.", score:20, quality:"good",
           reason:"Réponse exemplaire : plan structuré et précis, vous intégrez sa commande d'aliment hydrolysé (valorisation de son initiative), vous proposez une alternative thérapeutique moderne aux corticoïdes (oclacitinib), vous donnez un horizon temporel clair. Le 'sachant' apprécie la précision et les plans concrets."},
          {text:"On fait d'abord le bilan, et on verra ensuite.", score:-5, quality:"bad",
           reason:"Vague. Après trois échanges intenses avec un 'sachant', un plan flou est perçu comme une absence de compétence ou d'implication. Ce profil a besoin d'un plan détaillé et d'un calendrier."},
          {text:"Je propose un examen complet maintenant, et si tout est normal, on peut envisager le traitement que vous suggérez.", score:8, quality:"warn",
           reason:"Correct mais vous semblez conditionner votre plan à ce qu'il a dit, ce qui cède partiellement à sa pression. Mieux vaut présenter votre propre démarche structurée indépendamment."},
          {text:"On va d'abord calmer la démangeaison avec un traitement symptomatique, et parallèlement mettre en place la démarche diagnostique.", score:12, quality:"warn",
           reason:"Approche pragmatique valide. Mais elle manque de détail sur la démarche diagnostique, ce qui est insuffisant pour un profil 'sachant' qui demande de la précision."}
        ]
      }
    ],
    finalTip:"Profil SACHANT : Ne jamais confronter directement. Utiliser la technique du 'oui… et' (valider ses données → montrer leur limite → proposer mieux). Parler son langage (corrélation, biais, risque individuel). Il devient un allié quand vous le respectez intellectuellement."
  },

  // ────────────────────────────────────────────────────────
  //  CAS 3 : L'AGRESSIF / IMPATIENT
  // ────────────────────────────────────────────────────────
  {
    id:2, name:"M. Garnier", animal:"Rocky (Boxer, 5 ans)", emoji:"😤",
    pet:"chien", petEmoji:"🐕",
    color:"#dc2626", badgeColor:"#fee2e2", badgeText:"😤 Agressif/Impatient", badgeTC:"#991b1b",
    intro:"Ça fait une heure que j'attends dans cette salle ! Mon chien a une patte qui saigne et on m'a dit 'd'attendre votre tour'. C'est n'importe quoi ! Dans la vraie vie, une urgence ça passe avant les rendez-vous routiniers, non ?! Je veux voir le responsable.",
    type:"aggressive",
    steps:[
      {
        ownerMsg:"Ne me dites pas 'je comprends', ça m'énerve quand les gens disent ça sans rien faire. Je veux savoir POURQUOI on a attendu une heure et ce que vous allez faire MAINTENANT pour Rocky.",
        options:[
          {text:"Je vous entends. Et je ne vais pas vous dire 'je comprends' — je vais agir. Montrez-moi la patte de Rocky immédiatement.", score:20, quality:"good",
           reason:"Brillant. Vous entendez sa demande explicite ('ne dites pas je comprends'), vous l'honorez littéralement, et vous passez immédiatement à l'action. C'est la technique de la 'réponse sur mesure' : adapter votre réponse au mot près à ce que le propriétaire vient de vous dire. L'action immédiate désamorce 80% de l'agressivité."},
          {text:"Monsieur, je comprends votre frustration mais je dois vous demander de rester calme sinon je ne pourrai pas vous recevoir.", score:-15, quality:"bad",
           reason:"Double erreur fatale : utiliser 'je comprends' qu'il vient d'interdire, et menacer de ne pas le recevoir. Vous escaladez le conflit et prenez le risque d'un incident. Ne jamais conditionner les soins à un comportement du propriétaire."},
          {text:"L'attente est due à plusieurs urgences qui sont passées avant. Je suis là maintenant — montrez-moi Rocky.", score:8, quality:"warn",
           reason:"Se justifier sur l'attente est souvent perçu comme une excuse. Avec un profil agressif, les justifications n'apaisent pas — seule l'action compte. La deuxième partie est bonne."},
          {text:"Je suis vraiment désolé pour cette attente, c'est inacceptable, vous avez raison.", score:-8, quality:"bad",
           reason:"L'auto-flagellation extrême face à l'agressivité peut être perçue comme de la faiblesse ou de l'insincérité. Elle n'apaise pas non plus — elle peut au contraire encourager à aller plus loin dans les exigences."}
        ]
      },
      {
        ownerMsg:"Il s'est coupé sur un grillage ce matin. Il saigne encore un peu. Et là vous regardez sa patte depuis deux minutes — qu'est-ce que vous attendez pour faire quelque chose ?!",
        options:[
          {text:"Je nettoie la plaie maintenant pendant qu'on parle. La coupure est superficielle, elle ne touche pas les tendons — bonne nouvelle. Je vais faire une compression et vous expliquer la suite.", score:20, quality:"good",
           reason:"Vous agissez pendant que vous parlez — c'est exactement ce qu'il faut avec un impatient. Le multi-tâche (soin + information simultanés) montre de la compétence et de la réactivité. La 'bonne nouvelle' interposée diminue la tension."},
          {text:"Je dois d'abord évaluer correctement avant d'agir, vous comprendrez que la précipitation en médecine c'est dangereux.", score:-10, quality:"bad",
           reason:"Ce discours pédagogique est parfaitement adapté en consultation tranquille — il est totalement contre-productif avec un profil agressif et impatient. Vous semblez le faire attendre encore et lui donner une leçon en prime."},
          {text:"Je suis en train de regarder pour ne pas rater quelque chose d'important. Encore trente secondes.", score:8, quality:"warn",
           reason:"Vous expliquez votre démarche, c'est bien. Mais 'trente secondes' peut sembler une promesse arbitraire. Mieux vaut décrire ce que vous faites en temps réel, ce qui montre l'activité plutôt que l'attente."},
          {text:"Rocky est en bonne main. Plaie superficielle, pas de tendon touché. Je nettoie.", score:14, quality:"good",
           reason:"Efficace et rassurant. Réponse courte et action directe, bien adaptée à ce profil. Légèrement moins excellent car vous n'associez pas le propriétaire à la démarche."}
        ]
      },
      {
        ownerMsg:"Bien... Alors c'est pas grave ? Parce que j'ai annulé une réunion pour ça. Et là vous allez me sortir une facture de 200 balles pour une égratignure ?",
        options:[
          {text:"La plaie nécessite un nettoyage antiseptique soigneux, un pansement protecteur et une ordonnance antibiotique préventive — c'est ce qui évitera une infection sur une plaie par grillage potentiellement rouillé. Je vais vous détailler exactement ce qui est facturé et pourquoi.", score:20, quality:"good",
           reason:"Réponse transparente et professionnelle : vous justifiez chaque acte cliniquement (risque infectieux sur grillage = argument solide), vous anticipez la question de la facture, et vous proposez la transparence. Avec un profil agressif, la transparence financière anticipée désamorce les conflits post-consultation."},
          {text:"Le prix de la consultation est fixé par le cabinet, je n'y peux rien.", score:-15, quality:"bad",
           reason:"Dégager sa responsabilité sur 'le cabinet' est perçu comme du désengagement et de la mauvaise foi. C'est la pire réponse possible sur la question financière — elle confirme son sentiment d'être mal traité."},
          {text:"Je comprends. On va faire au plus simple pour minimiser les coûts tout en soignant correctement Rocky.", score:5, quality:"warn",
           reason:"L'intention est bonne mais 'faire au plus simple' peut suggérer des soins au rabais. Et vous utilisez 'je comprends' qu'il a explicitement rejeté plus tôt."},
          {text:"Une blessure par grillage c'est à risque tétanique et infectieux — ça mérite des soins sérieux. Mais je vais vous expliquer exactement ce que je prescris et pourquoi, et vous décidez en connaissance de cause.", score:16, quality:"good",
           reason:"Bonne réponse : vous posez le cadre médical (risque objectif) et vous rendez le propriétaire décideur éclairé. L'autonomie décisionnelle apaise souvent les profils contrôlants et agressifs."}
        ]
      }
    ],
    finalTip:"Profil AGRESSIF : La règle d'or est l'action immédiate. L'impatient ne veut pas d'explications — il veut voir qu'on agit. Adapter votre réponse mot pour mot à ses demandes explicites ('ne dites pas je comprends'). La transparence financière anticipée évite les conflits."
  },

  // ────────────────────────────────────────────────────────
  //  CAS 4 : LE NÉGLIGENT / DANS LE DÉNI
  // ────────────────────────────────────────────────────────
  {
    id:3, name:"Mme Renard", animal:"Gribouille (chien, 12 ans)", emoji:"😶",
    pet:"chien", petEmoji:"🐕",
    color:"#7c3aed", badgeColor:"#ede9fe", badgeText:"😶 Dans le déni", badgeTC:"#5b21b6",
    intro:"Bonjour. Gribouille tousse un peu depuis... je sais pas, deux ou trois mois. C'est probablement les pollens ou le changement de saison. Mon ancien vétérinaire disait toujours qu'il était costaud. À son âge, c'est normal d'avoir des petits trucs non ?",
    type:"denial",
    steps:[
      {
        ownerMsg:"Bon il mange encore bien — moins qu'avant, mais il mange. Et il se fatigue vite, mais c'est l'âge. 12 ans c'est vieux pour un chien, non ? Mon père aussi il se fatigue plus vite et on lui dit pas qu'il est malade.",
        options:[
          {text:"Vous avez raison, le vieillissement explique beaucoup de choses. On va quand même jeter un œil.", score:-12, quality:"bad",
           reason:"Valider le raisonnement de minimisation d'un propriétaire dont l'animal a une toux productive depuis 3 mois avec perte d'appétit et intolérance à l'effort est médicalement dangereux. Ces trois symptômes associés chez un senior constituent un tableau clinique qui justifie une investigation urgente."},
          {text:"12 ans pour un chien c'est effectivement un âge qui mérite attention. Je vais être direct avec vous : trois mois de toux, une diminution de l'appétit et une fatigue à l'effort chez un chien de cet âge, ce n'est pas 'l'âge' — c'est un tableau qui mérite un bilan. Je veux m'assurer qu'on ne passe pas à côté de quelque chose que l'on peut traiter.", score:20, quality:"good",
           reason:"Réponse modèle avec un profil déni/négligent : vous êtes direct sans être brutal ('je vais être direct'), vous listez les symptômes de façon précise pour montrer leur accumulation, et vous ouvrez une porte positive ('quelque chose qu'on peut traiter'). Le bénéfice potentiel est la meilleure motivation pour un propriétaire dans le déni."},
          {text:"Ces symptômes peuvent être sérieux, il faut absolument faire une radio thoracique et une prise de sang en urgence.", score:5, quality:"warn",
           reason:"Médicalement justifié mais trop directif. Avec un propriétaire dans le déni, l'urgentisation sans explication du raisonnement produit souvent un rejet ou une fuite. Il faut d'abord que le propriétaire comprenne pourquoi."},
          {text:"La comparaison avec votre père est intéressante. Mais si votre père toussait depuis 3 mois et se fatiguait progressivement, son médecin ferait un bilan, non ?", score:14, quality:"good",
           reason:"Bonne technique de 'retournement de métaphore' : vous utilisez sa propre comparaison pour l'amener à votre conclusion. Créatif et efficace avec ce profil. Légèrement moins direct sur les enjeux médicaux concrets."}
        ]
      },
      {
        ownerMsg:"Vous pensez que c'est grave ? Parce que... bon, j'avoue que j'ai un peu tardé. Je me disais que ça allait passer. Et puis les radios, la prise de sang... ça coûte cher et à son âge, si c'est grave, qu'est-ce que ça changera ?",
        options:[
          {text:"Je comprends votre questionnement. Mais je vais vous poser une question directe : si Gribouille souffre et qu'on peut le soulager, est-ce qu'on le fait ou pas ? C'est ça la vraie question, indépendamment du pronostic.", score:20, quality:"good",
           reason:"Réponse pivot : vous reconnaissez le questionnement légitime (coût + âge + pronostic), mais vous recentrez sur l'enjeu éthique fondamental : le confort de l'animal. Cette question directe est un outil puissant pour mobiliser un propriétaire dans le déni qui finalement exprime surtout de la culpabilité différée."},
          {text:"L'âge n'est pas une raison pour ne pas traiter. Vous avez une responsabilité envers Gribouille.", score:-10, quality:"bad",
           reason:"La culpabilisation explicite avec un propriétaire qui vient d'admettre avoir tardé produit de la honte, pas de la mobilisation. La honte génère souvent le retrait, pas l'action. Vous risquez de le perdre."},
          {text:"Les examens nous permettront d'orienter le traitement. Même pour un chien âgé, beaucoup de pathologies sont traitables avec une bonne qualité de vie.", score:14, quality:"good",
           reason:"Bon message d'espoir thérapeutique. Manque la prise en compte directe de la question du coût et du 'à quoi bon'."},
          {text:"Bien sûr que ça changera quelque chose. Un chien de 12 ans peut encore avoir 2-3 ans devant lui si on agit maintenant.", score:8, quality:"warn",
           reason:"L'argument de l'espérance de vie est parfois efficace, mais il peut sembler une pression. Sans savoir le diagnostic, promettre 2-3 ans est risqué médicalement et peut créer une attente non tenue."}
        ]
      },
      {
        ownerMsg:"...Vous avez raison. Si lui il souffre... je veux pas qu'il souffre. C'est juste que j'ai peur d'entendre quelque chose que je pourrai pas gérer. Gribouille c'est toute ma vie depuis 12 ans.",
        options:[
          {text:"Je vous entends. Ce que vous décrivez, c'est l'amour d'un propriétaire pour son animal. Et cet amour, c'est exactement la raison pour laquelle on va regarder la vérité en face ensemble — parce que Gribouille mérite qu'on sache ce qu'il a. Quelle que soit la réponse, vous ne serez pas seule pour y faire face.", score:20, quality:"good",
           reason:"Réponse d'une qualité émotionnelle rare : vous nommez et valorisez l'attachement sans le juger, vous repositionnez le courage du diagnostic comme un acte d'amour envers Gribouille, et vous offrez un soutien explicite. Cette réponse est la clé pour transformer un propriétaire dans le déni en partenaire de soin actif."},
          {text:"C'est normal d'avoir peur. Mais l'ignorance ne protège pas Gribouille, elle le prive peut-être d'un traitement.", score:8, quality:"warn",
           reason:"Vrai mais un peu froid. 'L'ignorance ne protège pas' est un argument rationnel là où le propriétaire est dans un registre émotionnel intense. Il faut répondre émotion par émotion avant d'amener la logique."},
          {text:"Je comprends. Sachez que quoi qu'il arrive, on a des solutions pour le confort de Gribouille.", score:12, quality:"warn",
           reason:"Message de soutien correct mais générique. Il ne reconnaît pas spécifiquement l'intensité du lien décrit ('toute ma vie depuis 12 ans'). Une réponse aussi personnelle méritait une réponse personnalisée."},
          {text:"On va faire les examens, et ensuite on prend les décisions au fur et à mesure. Étape par étape.", score:14, quality:"good",
           reason:"La progression par étapes est rassurante pour quelqu'un submergé émotionnellement. Bonne réponse, mais elle passe à côté de la dimension affective très forte de ce moment de la consultation."}
        ]
      }
    ],
    finalTip:"Profil DÉNI/NÉGLIGENT : L'outil clé est de distinguer déni par indifférence (rare) et déni par peur (fréquent). Ici : peur du verdict + sentiment de culpabilité. Jamais de culpabilisation. Recentrer sur le confort de l'animal comme motivation universelle. Offrir un accompagnement explicite ('vous ne serez pas seul')."
  }
];

// ═══════════════════════════════════════════════════════════
//  VARIABLES GLOBALES
// ═══════════════════════════════════════════════════════════
let gameState = "title";
let totalScore = 0;
let consultedOwners = [];
let currentOwner = null;
let currentStep = 0;
let stepScores = [];
let consultLog = [];
let nearbyOwner = null;
const keys = {};
let playerX = 0, playerZ = 3;
let animFrame = 0;
const ownerMeshes = [];
let vetGroup = null;

// ═══════════════════════════════════════════════════════════
//  THREE.JS SETUP
// ═══════════════════════════════════════════════════════════
const canvas = document.getElementById('c');
const renderer = new THREE.WebGLRenderer({canvas, antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.setClearColor(0xbfdbf7);

const scene = new THREE.Scene();
scene.fog = new THREE.Fog(0xbfdbf7, 18, 38);

const camera = new THREE.PerspectiveCamera(68, window.innerWidth/window.innerHeight, 0.1, 100);
camera.position.set(0, 3, 7);

// Lumières soignées
const ambient = new THREE.AmbientLight(0xfff5e4, 0.55);
scene.add(ambient);
const sun = new THREE.DirectionalLight(0xfff8f0, 0.9);
sun.position.set(6, 12, 8);
sun.castShadow = true;
sun.shadow.mapSize.width = 1024;
sun.shadow.mapSize.height = 1024;
scene.add(sun);
const fill = new THREE.DirectionalLight(0xe0f0ff, 0.35);
fill.position.set(-6, 4, -6);
scene.add(fill);

// ═══════════════════════════════════════════════════════════
//  CONSTRUCTION SALLE D'ATTENTE
// ═══════════════════════════════════════════════════════════
function buildRoom() {
  // Sol carrelé
  const floorMat = new THREE.MeshLambertMaterial({color:0xf5f0eb});
  const floor = new THREE.Mesh(new THREE.PlaneGeometry(28,24), floorMat);
  floor.rotation.x = -Math.PI/2; floor.receiveShadow = true;
  scene.add(floor);

  // Carreaux alternés
  const t1 = new THREE.MeshLambertMaterial({color:0xfaf5f0});
  const t2 = new THREE.MeshLambertMaterial({color:0xeee8e0});
  for(let x=-6;x<=6;x+=2) for(let z=-6;z<=6;z+=2) {
    const tile = new THREE.Mesh(new THREE.PlaneGeometry(1.95,1.95), (x+z)%4===0?t1:t2);
    tile.rotation.x = -Math.PI/2; tile.position.set(x,0.01,z);
    scene.add(tile);
  }

  // Murs
  const wallM = new THREE.MeshLambertMaterial({color:0xedf3f8});
  const wallAccent = new THREE.MeshLambertMaterial({color:0xd4e6f4});
  // Mur du fond
  const wb = new THREE.Mesh(new THREE.BoxGeometry(22,6,0.25), wallM);
  wb.position.set(0,3,-9); scene.add(wb);
  // Liseré couleur
  const stripe = new THREE.Mesh(new THREE.BoxGeometry(22,0.5,0.26), wallAccent);
  stripe.position.set(0,1.5,-8.88); scene.add(stripe);
  const wallL = new THREE.Mesh(new THREE.BoxGeometry(0.25,6,22), wallM);
  wallL.position.set(-9,3,0); scene.add(wallL);
  const wallR = new THREE.Mesh(new THREE.BoxGeometry(0.25,6,22), wallM);
  wallR.position.set(9,3,0); scene.add(wallR);
  const ceil = new THREE.Mesh(new THREE.PlaneGeometry(22,22), new THREE.MeshLambertMaterial({color:0xfafafa}));
  ceil.rotation.x = Math.PI/2; ceil.position.set(0,6,0); scene.add(ceil);

  // Enseigne cabinet
  const enseigneBack = new THREE.Mesh(new THREE.BoxGeometry(6,1.2,0.15), new THREE.MeshLambertMaterial({color:0x1a6b8a}));
  enseigneBack.position.set(0,4.8,-8.85); scene.add(enseigneBack);
  const cross = new THREE.Mesh(new THREE.BoxGeometry(0.6,0.15,0.2), new THREE.MeshLambertMaterial({color:0xffffff}));
  cross.position.set(-2.2,4.8,-8.78); scene.add(cross);
  const crossV2 = new THREE.Mesh(new THREE.BoxGeometry(0.15,0.6,0.2), new THREE.MeshLambertMaterial({color:0xffffff}));
  crossV2.position.set(-2.2,4.8,-8.78); scene.add(crossV2);

  // Bancs
  const benchW = new THREE.MeshLambertMaterial({color:0xa0522d});
  const benchMetal = new THREE.MeshLambertMaterial({color:0x555555});
  [[-5,-4.5],[-2,-4.5],[1,-4.5],[-5,-2.5]].forEach(([x,z]) => {
    const seat = new THREE.Mesh(new THREE.BoxGeometry(1.8,0.12,0.55), benchW);
    seat.position.set(x,0.5,z); seat.castShadow = true; scene.add(seat);
    const back = new THREE.Mesh(new THREE.BoxGeometry(1.8,0.55,0.08), benchW);
    back.position.set(x,0.82,z-0.23); scene.add(back);
    [[-0.75,0.2],[0.75,0.2],[-0.75,-0.2],[0.75,-0.2]].forEach(([dx,dz]) => {
      const leg = new THREE.Mesh(new THREE.CylinderGeometry(0.05,0.05,0.48,6), benchMetal);
      leg.position.set(x+dx,0.25,z+dz); scene.add(leg);
    });
  });

  // Table basse
  const tableTop = new THREE.Mesh(new THREE.BoxGeometry(1,0.06,0.65), new THREE.MeshLambertMaterial({color:0xd4a76a}));
  tableTop.position.set(-1.5,0.4,-5.5); scene.add(tableTop);
  // Magazines
  const mag = new THREE.Mesh(new THREE.BoxGeometry(0.3,0.03,0.22), new THREE.MeshLambertMaterial({color:0xf87171}));
  mag.position.set(-1.5,0.44,-5.5); scene.add(mag);

  // Comptoir réception
  const counterM = new THREE.MeshLambertMaterial({color:0x2563eb});
  const counter = new THREE.Mesh(new THREE.BoxGeometry(5,1.05,0.9), counterM);
  counter.position.set(0,0.52,6.5); scene.add(counter);
  const cTop = new THREE.Mesh(new THREE.BoxGeometry(5.2,0.08,1.1), new THREE.MeshLambertMaterial({color:0x1d4ed8}));
  cTop.position.set(0,1.08,6.5); scene.add(cTop);
  // Écran
  const monBase = new THREE.Mesh(new THREE.CylinderGeometry(0.1,0.1,0.04,8), new THREE.MeshLambertMaterial({color:0x333}));
  monBase.position.set(-1,1.16,6.2); scene.add(monBase);
  const monScreen = new THREE.Mesh(new THREE.BoxGeometry(0.9,0.6,0.04), new THREE.MeshLambertMaterial({color:0x1a1a2e}));
  monScreen.position.set(-1,1.5,6.2); scene.add(monScreen);
  const monGlow = new THREE.Mesh(new THREE.BoxGeometry(0.82,0.52,0.02), new THREE.MeshLambertMaterial({color:0x3b82f6,emissive:0x1d4ed8,emissiveIntensity:0.5}));
  monGlow.position.set(-1,1.5,6.22); scene.add(monGlow);

  // Plantes
  [[7.5,0,-8],[-7.5,0,-8],[7.5,0,4],[-7.5,0,4]].forEach(([x,y,z]) => {
    const pot = new THREE.Mesh(new THREE.CylinderGeometry(0.22,0.28,0.42,10), new THREE.MeshLambertMaterial({color:0xb45309}));
    pot.position.set(x,0.21,z); scene.add(pot);
    const plant = new THREE.Mesh(new THREE.SphereGeometry(0.55,8,6), new THREE.MeshLambertMaterial({color:0x16a34a}));
    plant.position.set(x,0.85,z); plant.scale.set(1,1.1,1); scene.add(plant);
    const plantS = new THREE.Mesh(new THREE.SphereGeometry(0.38,8,5), new THREE.MeshLambertMaterial({color:0x15803d}));
    plantS.position.set(x+0.28,0.75,z+0.1); scene.add(plantS);
  });

  // Tableau info patients
  const board = new THREE.Mesh(new THREE.BoxGeometry(3.5,2.2,0.1), new THREE.MeshLambertMaterial({color:0x164e63}));
  board.position.set(6.5,2.8,-8.85); scene.add(board);

  // Lumières plafond
  for(let x=-4;x<=4;x+=4) for(let z=-4;z<=4;z+=4) {
    const lamp = new THREE.Mesh(new THREE.CylinderGeometry(0.35,0.35,0.08,16), new THREE.MeshLambertMaterial({color:0xfffde0,emissive:0xfff5a0,emissiveIntensity:0.6}));
    lamp.position.set(x,5.95,z); scene.add(lamp);
    const pt = new THREE.PointLight(0xfff8e7, 0.45, 10);
    pt.position.set(x,5.6,z); scene.add(pt);
  }
}

// ═══════════════════════════════════════════════════════════
//  PERSONNAGES STYLE PIXAR (tête ronde, grands yeux)
// ═══════════════════════════════════════════════════════════
function makeChar(owner, px, pz) {
  const g = new THREE.Group();
  const col = parseInt(owner.color.slice(1),16);

  // Jambes
  const legM = new THREE.MeshLambertMaterial({color:0x1e293b});
  [-0.16,0.16].forEach(lx => {
    const leg = new THREE.Mesh(new THREE.CylinderGeometry(0.1,0.09,0.62,8), legM);
    leg.position.set(lx,0.31,0); leg.castShadow = true; g.add(leg);
    const shoe = new THREE.Mesh(new THREE.SphereGeometry(0.13,8,6), new THREE.MeshLambertMaterial({color:0x0f172a}));
    shoe.scale.set(1,0.55,1.35); shoe.position.set(lx,0.03,0.07); g.add(shoe);
  });

  // Corps arrondi Pixar
  const bodyGeo = new THREE.SphereGeometry(0.33,10,8);
  const body = new THREE.Mesh(bodyGeo, new THREE.MeshLambertMaterial({color:col}));
  body.scale.set(1,1.35,0.88); body.position.y = 0.88; body.castShadow = true; g.add(body);

  // Col / cou
  const neck = new THREE.Mesh(new THREE.CylinderGeometry(0.11,0.13,0.22,8), new THREE.MeshLambertMaterial({color:0xfde3c2}));
  neck.position.y = 1.35; g.add(neck);

  // Bras
  const armM = new THREE.MeshLambertMaterial({color:col});
  [-1,1].forEach(side => {
    const armGeo = new THREE.CylinderGeometry(0.09,0.07,0.5,8);
    const arm = new THREE.Mesh(armGeo, armM);
    arm.position.set(side*0.46, 0.88, 0);
    arm.rotation.z = side * -0.22;
    arm.castShadow = true; g.add(arm);
    const hand = new THREE.Mesh(new THREE.SphereGeometry(0.1,8,6), new THREE.MeshLambertMaterial({color:0xfde3c2}));
    hand.position.set(side*0.52, 0.6, 0); g.add(hand);
  });

  // Tête Pixar (grande, arrondie, expressive)
  const headGeo = new THREE.SphereGeometry(0.4,12,10);
  const head = new THREE.Mesh(headGeo, new THREE.MeshLambertMaterial({color:0xfde3c2}));
  head.scale.set(1,1.1,0.95); head.position.y = 1.78; head.castShadow = true; g.add(head);
  g.userData.head = head;

  // Cheveux (différents par ID)
  const hairCols = [0x7c2d12, 0x1e3a5f, 0x3d1a00, 0x1a1a1a];
  const hairM = new THREE.MeshLambertMaterial({color: hairCols[owner.id % hairCols.length]});
  if(owner.id % 2 === 0) { // féminin : cheveux longs
    const hMain = new THREE.Mesh(new THREE.SphereGeometry(0.43,10,8), hairM);
    hMain.position.y = 1.87; hMain.scale.set(1,0.65,1); g.add(hMain);
    [-0.28,0.28].forEach(hx => {
      const hSide = new THREE.Mesh(new THREE.SphereGeometry(0.22,8,6), hairM);
      hSide.position.set(hx,1.7,0); g.add(hSide);
    });
    const hBack = new THREE.Mesh(new THREE.SphereGeometry(0.3,8,6), hairM);
    hBack.position.set(0,1.6,-0.2); g.add(hBack);
  } else { // masculin : cheveux courts
    const hTop = new THREE.Mesh(new THREE.SphereGeometry(0.42,10,8), hairM);
    hTop.position.y = 1.9; hTop.scale.set(1,0.35,1); g.add(hTop);
  }

  // Yeux grands style Pixar
  const whiteM = new THREE.MeshLambertMaterial({color:0xffffff});
  const darkM = new THREE.MeshLambertMaterial({color:0x1a1a2e});
  const glowM = new THREE.MeshLambertMaterial({color:0xffffff,emissive:0xffffff,emissiveIntensity:0.3});
  [-0.14,0.14].forEach(ex => {
    const ew = new THREE.Mesh(new THREE.SphereGeometry(0.1,10,8), whiteM);
    ew.position.set(ex,1.8,0.36); ew.scale.set(1,1.12,0.6); g.add(ew);
    const pupil = new THREE.Mesh(new THREE.SphereGeometry(0.062,8,6), darkM);
    pupil.position.set(ex,1.8,0.38); g.add(pupil);
    const iris = new THREE.Mesh(new THREE.SphereGeometry(0.04,6,4), new THREE.MeshLambertMaterial({color:0x3b82f6}));
    iris.position.set(ex,1.8,0.385); g.add(iris);
    const shine = new THREE.Mesh(new THREE.SphereGeometry(0.022,6,4), glowM);
    shine.position.set(ex+0.025,1.825,0.4); g.add(shine);
  });

  // Sourcils expressifs selon personnalité
  const browM = new THREE.MeshLambertMaterial({color: hairCols[owner.id % hairCols.length]});
  [-0.14,0.14].forEach((bx,i) => {
    const brow = new THREE.Mesh(new THREE.BoxGeometry(0.13,0.03,0.04), browM);
    let rz = 0;
    if(owner.type==='aggressive') rz = bx>0 ? 0.35 : -0.35;
    else if(owner.type==='panicky') rz = bx>0 ? -0.32 : 0.32;
    else if(owner.type==='knowitall') rz = bx>0 ? 0.18 : -0.18;
    else rz = 0; // déni : plat
    brow.position.set(bx, 1.92, 0.35); brow.rotation.z = rz; g.add(brow);
  });

  // Nez
  const nose = new THREE.Mesh(new THREE.SphereGeometry(0.05,8,6), new THREE.MeshLambertMaterial({color:0xf9b98a}));
  nose.position.set(0,1.73,0.42); g.add(nose);

  // Bouche
  const mouthM = new THREE.MeshLambertMaterial({color:0xd1626d});
  if(owner.type==='aggressive') {
    const mGeo = new THREE.TorusGeometry(0.07,0.018,8,12,Math.PI);
    const m = new THREE.Mesh(mGeo, mouthM);
    m.position.set(0,1.65,0.37); m.rotation.x = Math.PI; g.add(m); // bouche pincée
  } else if(owner.type==='panicky') {
    const mGeo = new THREE.TorusGeometry(0.07,0.018,8,12,Math.PI);
    const m = new THREE.Mesh(mGeo, mouthM);
    m.position.set(0,1.65,0.37); g.add(m); // bouche ouverte
  } else {
    const mGeo = new THREE.TorusGeometry(0.06,0.015,8,10,Math.PI*0.5);
    const m = new THREE.Mesh(mGeo, mouthM);
    m.position.set(-0.03,1.67,0.38); g.add(m); // neutre
  }

  // Cage / sac transport animal
  const cageCol = owner.pet==='chat' ? 0xf97316 : 0x3b82f6;
  const cage = new THREE.Mesh(new THREE.BoxGeometry(0.52,0.46,0.52),
    new THREE.MeshLambertMaterial({color:cageCol, transparent:true, opacity:0.35}));
  cage.position.set(0.6,0.34,0.05); g.add(cage);
  for(let b=0;b<5;b++) {
    const bar = new THREE.Mesh(new THREE.CylinderGeometry(0.012,0.012,0.46,4),
      new THREE.MeshLambertMaterial({color:0x888888}));
    bar.position.set(0.6+(b/4-0.5)*0.44, 0.34, 0.31); g.add(bar);
  }
  const handle = new THREE.Mesh(new THREE.TorusGeometry(0.12,0.02,6,12),
    new THREE.MeshLambertMaterial({color:0x666666}));
  handle.position.set(0.6,0.68,0.05); handle.rotation.x = Math.PI/2; g.add(handle);

  g.position.set(px, 0, pz);
  g.userData = { owner, animOffset: Math.random()*Math.PI*2, bobFreq: 0.035 + Math.random()*0.01 };
  scene.add(g);
  ownerMeshes.push(g);
  return g;
}

// Vétérinaire joueur
function makeVet() {
  const g = new THREE.Group();
  // Corps blouse blanche
  const bodyGeo = new THREE.SphereGeometry(0.3,10,8);
  const body = new THREE.Mesh(bodyGeo, new THREE.MeshLambertMaterial({color:0xfefefe}));
  body.scale.set(1,1.3,0.9); body.position.y = 0.85; body.castShadow = true; g.add(body);
  // Poche + croix
  const pocket = new THREE.Mesh(new THREE.BoxGeometry(0.15,0.12,0.04), new THREE.MeshLambertMaterial({color:0xe0e0e0}));
  pocket.position.set(-0.2,0.95,0.28); g.add(pocket);
  const crossH = new THREE.Mesh(new THREE.BoxGeometry(0.18,0.04,0.05), new THREE.MeshLambertMaterial({color:0x22c55e}));
  const crossV3 = new THREE.Mesh(new THREE.BoxGeometry(0.04,0.18,0.05), new THREE.MeshLambertMaterial({color:0x22c55e}));
  crossH.position.set(0.15,0.98,0.3); crossV3.position.set(0.15,0.98,0.3); g.add(crossH,crossV3);
  // Jambes
  [-0.14,0.14].forEach(lx => {
    const l = new THREE.Mesh(new THREE.CylinderGeometry(0.1,0.08,0.58,8), new THREE.MeshLambertMaterial({color:0x2c3e50}));
    l.position.set(lx,0.29,0); g.add(l);
  });
  // Cou + tête
  const neck = new THREE.Mesh(new THREE.CylinderGeometry(0.1,0.12,0.2,8), new THREE.MeshLambertMaterial({color:0xfde3c2}));
  neck.position.y = 1.32; g.add(neck);
  const head = new THREE.Mesh(new THREE.SphereGeometry(0.34,12,10), new THREE.MeshLambertMaterial({color:0xfde3c2}));
  head.scale.set(1,1.1,1); head.position.y = 1.7; head.castShadow = true; g.add(head);
  // Yeux
  [-0.12,0.12].forEach(ex => {
    const e = new THREE.Mesh(new THREE.SphereGeometry(0.055,8,6), new THREE.MeshLambertMaterial({color:0x1a1a2e}));
    e.position.set(ex,1.73,0.3); g.add(e);
    const s = new THREE.Mesh(new THREE.SphereGeometry(0.02,6,4), new THREE.MeshLambertMaterial({color:0xffffff,emissive:0xffffff,emissiveIntensity:0.5}));
    s.position.set(ex+0.02,1.75,0.32); g.add(s);
  });
  // Stéthoscope
  const steth = new THREE.Mesh(new THREE.TorusGeometry(0.16,0.018,8,18), new THREE.MeshLambertMaterial({color:0x0f172a}));
  steth.position.set(0,1.2,0.25); steth.rotation.x = -0.35; g.add(steth);
  const stethEnd = new THREE.Mesh(new THREE.SphereGeometry(0.04,8,6), new THREE.MeshLambertMaterial({color:0x334155}));
  stethEnd.position.set(0,0.98,0.32); g.add(stethEnd);

  g.position.set(0,0,3);
  scene.add(g);
  return g;
}

// ═══════════════════════════════════════════════════════════
//  GAME LOGIC
// ═══════════════════════════════════════════════════════════
const ownerPositions = [[-5,0,-4.3],[-2,0,-4.3],[1,0,-4.3],[-5,0,-2.3]];

function startGame() {
  document.getElementById('title-screen').style.display = 'none';
  document.getElementById('hud').style.display = 'flex';
  document.getElementById('controls-hint').style.display = 'flex';
  gameState = 'waiting';
  buildRoom();
  OWNERS.forEach((o,i) => makeChar(o, ownerPositions[i][0], ownerPositions[i][2]));
  vetGroup = makeVet();
  playerX = 0; playerZ = 3;
  totalScore = 0; consultedOwners = []; consultLog = []; stepScores = [];
  updateHUD();
  animate();
}

function animate() {
  if(gameState==='finished') return;
  requestAnimationFrame(animate);
  animFrame++;

  if(gameState==='waiting') {
    move(); detectNear(); animChars();
  }

  // Caméra smooth follow
  const tx = playerX, tz = playerZ + 3.8, ty = 3.2;
  camera.position.x += (tx - camera.position.x) * 0.1;
  camera.position.z += (tz - camera.position.z) * 0.1;
  camera.position.y += (ty - camera.position.y) * 0.1;
  camera.lookAt(playerX, 1.2, playerZ);

  if(vetGroup) {
    vetGroup.position.x = playerX;
    vetGroup.position.z = playerZ;
    vetGroup.position.y = Math.sin(animFrame*0.09)*0.04;
  }
  renderer.render(scene, camera);
}

function animChars() {
  ownerMeshes.forEach((m,i) => {
    if(consultedOwners.includes(i)) { m.visible=false; return; }
    const t = animFrame * m.userData.bobFreq + m.userData.animOffset;
    m.position.y = Math.sin(t) * 0.05;
    if(m.userData.owner.type==='panicky') m.rotation.z = Math.sin(t*2.8)*0.025;
    if(m.userData.owner.type==='aggressive') {
      m.position.x = ownerPositions[i][0] + Math.sin(t*2)*0.03;
    }
    const dx = playerX - m.position.x, dz = playerZ - m.position.z;
    m.rotation.y += (Math.atan2(dx,dz) - m.rotation.y) * 0.04;
  });
}

function move() {
  const sp = 0.085;
  if(keys['arrowup']||keys['z']||keys['w']) playerZ -= sp;
  if(keys['arrowdown']||keys['s']) playerZ += sp;
  if(keys['arrowleft']||keys['q']||keys['a']) playerX -= sp;
  if(keys['arrowright']||keys['d']) playerX += sp;
  playerX = Math.max(-8,Math.min(8,playerX));
  playerZ = Math.max(-8,Math.min(6.5,playerZ));
  if(vetGroup) {
    const dx = playerX-vetGroup.position.x, dz = playerZ-vetGroup.position.z;
    if(Math.abs(dx)+Math.abs(dz)>0.01) vetGroup.rotation.y = Math.atan2(dx,dz);
  }
}

function detectNear() {
  nearbyOwner = null;
  let minD = Infinity;
  ownerMeshes.forEach((m,i) => {
    if(consultedOwners.includes(i)) return;
    const d = Math.sqrt((playerX-m.position.x)**2+(playerZ-m.position.z)**2);
    if(d<2.8&&d<minD){ minD=d; nearbyOwner={mesh:m,index:i,owner:m.userData.owner}; }
  });
  const p = document.getElementById('interact-prompt');
  if(nearbyOwner) {
    p.style.display = 'block';
    p.innerHTML = `${nearbyOwner.owner.emoji} <strong>${nearbyOwner.owner.name}</strong> — Appuyer sur <kbd style="background:#4ecdc4;color:white;padding:2px 8px;border-radius:5px;font-weight:800">E</kbd>`;
  } else p.style.display = 'none';
}

// ═══════════════════════════════════════════════════════════
//  CONSULTATION
// ═══════════════════════════════════════════════════════════
function openConsult(owner, idx) {
  gameState = 'consulting';
  currentOwner = owner; currentStep = 0; stepScores = [];
  document.getElementById('interact-prompt').style.display = 'none';
  document.getElementById('c-avatar').textContent = owner.emoji;
  document.getElementById('c-avatar').style.background = owner.badgeColor;
  document.getElementById('c-name').textContent = owner.name;
  document.getElementById('c-animal').textContent = `${owner.petEmoji} ${owner.animal}`;
  const badge = document.getElementById('c-badge');
  badge.textContent = owner.badgeText;
  badge.style.background = owner.badgeColor;
  badge.style.color = owner.badgeTC;
  document.getElementById('dialogue-area').innerHTML = '';
  document.getElementById('options-area').innerHTML = '';
  updateProgress();
  addBubble(owner.name, owner.intro, 'owner');
  renderOpts(owner.steps[0].options);
  document.getElementById('consult-overlay').style.display = 'flex';
}

function updateProgress() {
  const pct = currentOwner ? (currentStep / currentOwner.steps.length) * 100 : 0;
  document.getElementById('progress-bar').style.width = pct + '%';
}

function addBubble(speaker, text, type) {
  const area = document.getElementById('dialogue-area');
  const d = document.createElement('div');
  d.className = `bubble bubble-${type}`;
  if(type==='owner') d.setAttribute('data-name', speaker);
  d.textContent = text;
  area.appendChild(d);
  setTimeout(() => d.scrollIntoView({behavior:'smooth',block:'end'}), 50);
}

function renderOpts(opts) {
  const area = document.getElementById('options-area');
  area.innerHTML = '';
  opts.forEach((o,i) => {
    const btn = document.createElement('button');
    btn.className = 'opt-btn';
    btn.innerHTML = `<span class="opt-key">${['A','B','C','D'][i]}</span><span>${o.text}</span>`;
    btn.onclick = () => pick(o, opts);
    area.appendChild(btn);
  });
}

async function pick(chosen, allOpts) {
  document.querySelectorAll('.opt-btn').forEach(b => b.disabled = true);
  addBubble('Vous', chosen.text, 'vet');
  await delay(350);
  showFB(chosen, allOpts);
  stepScores.push(chosen.score);
  totalScore += chosen.score;
  updateHUD();
  currentStep++;
  updateProgress();
  await delay(2000);
  if(currentStep < currentOwner.steps.length) {
    addBubble(currentOwner.name, currentOwner.steps[currentStep].ownerMsg, 'owner');
    renderOpts(currentOwner.steps[currentStep].options);
  } else {
    endConsult();
  }
}

function showFB(chosen, allOpts) {
  const area = document.getElementById('dialogue-area');
  const best = allOpts.reduce((a,b) => a.score>b.score?a:b);
  const cls = chosen.quality==='good' ? 'feedback-good' : chosen.quality==='warn' ? 'feedback-warn' : 'feedback-bad';
  const icon = chosen.quality==='good'?'✅':chosen.quality==='warn'?'⚠️':'❌';
  const label = chosen.quality==='good'?'Excellente réponse':chosen.quality==='warn'?'Réponse acceptable — peut mieux faire':'À améliorer';
  const pts = chosen.score>0?`+${chosen.score}`:String(chosen.score);
  const d = document.createElement('div');
  d.className = `feedback-box ${cls}`;
  d.innerHTML = `<div class="fb-score">${icon} ${pts} pts</div>
    <h4>${label}</h4>
    <p>${chosen.reason}</p>
    ${chosen.quality!=='good'?`<div class="best-answer">💡 <strong>Meilleure approche :</strong> "${best.text}"</div>`:''}`;
  area.appendChild(d);
  setTimeout(() => d.scrollIntoView({behavior:'smooth',block:'end'}), 50);
}

function endConsult() {
  const sum = stepScores.reduce((a,b)=>a+b,0);
  const maxP = currentOwner.steps.reduce((a,s) => a+Math.max(...s.options.map(o=>o.score)), 0);
  const pct = Math.round(Math.max(0,sum)/maxP*100);
  const area = document.getElementById('dialogue-area');
  const recap = document.createElement('div');
  recap.className = 'bubble bubble-system';
  recap.textContent = '— Fin de consultation —';
  area.appendChild(recap);
  const tip = document.createElement('div');
  tip.className = 'feedback-box feedback-good';
  tip.innerHTML = `<h4>📚 Bilan — ${currentOwner.name} · ${pct}% du score max</h4><p>${currentOwner.finalTip}</p>`;
  area.appendChild(tip);
  tip.scrollIntoView({behavior:'smooth'});
  const idx = OWNERS.findIndex(o=>o.id===currentOwner.id);
  consultedOwners.push(idx);
  consultLog.push({owner:currentOwner, score:sum, max:maxP, pct});
  const chip = document.createElement('div');
  chip.className = 'patient-chip';
  chip.innerHTML = `${currentOwner.emoji} ${currentOwner.name.split(' ')[1]} <span style="color:#4ecdc4;font-weight:700">${sum>0?'+':''}${sum}</span>`;
  document.getElementById('patients-done').appendChild(chip);
  updateHUD();
  const oArea = document.getElementById('options-area');
  oArea.innerHTML = '';
  const finBtn = document.createElement('button');
  finBtn.className = 'opt-btn';
  finBtn.style.cssText = 'border-color:#22c55e;background:#f0fdf4;font-weight:700';
  finBtn.innerHTML = `<span>${consultedOwners.length>=OWNERS.length?'🏆 Voir mes résultats':'✓ Retour en salle d\'attente'}</span>`;
  finBtn.onclick = () => {
    closeConsult();
    if(consultedOwners.length>=OWNERS.length) setTimeout(showResults,300);
  };
  oArea.appendChild(finBtn);
}

function closeConsult() {
  document.getElementById('consult-overlay').style.display = 'none';
  currentOwner = null;
  if(consultedOwners.length<OWNERS.length) gameState='waiting';
}

function updateHUD() {
  document.getElementById('score-hud').textContent = `⭐ ${totalScore} pts`;
  document.getElementById('patients-count').textContent = `${consultedOwners.length} / ${OWNERS.length} consultés`;
}

// ═══════════════════════════════════════════════════════════
//  RÉSULTATS FINAUX
// ═══════════════════════════════════════════════════════════
function showResults() {
  gameState = 'finished';
  document.getElementById('consult-overlay').style.display = 'none';
  const maxT = OWNERS.reduce((a,o)=>a+o.steps.reduce((b,s)=>b+Math.max(...s.options.map(x=>x.score)),0),0);
  const pct = Math.round(Math.max(0,totalScore)/maxT*100);
  let emoji,title,sub;
  if(pct>=85){emoji='🏆';title='Expert en communication';sub='Vous êtes prêt(e) pour la clinique';}
  else if(pct>=70){emoji='🌟';title='Très bon communicant';sub='Quelques situations à affiner';}
  else if(pct>=50){emoji='📈';title='Progression notable';sub='Des bases solides, continuez à pratiquer';}
  else{emoji='💪';title='Entraînement nécessaire';sub='La communication vétérinaire s\'apprend — recommencez !';}
  document.getElementById('res-emoji').textContent = emoji;
  document.getElementById('res-title').textContent = title;
  document.getElementById('res-sub').textContent = sub;
  document.getElementById('res-score').textContent = `${pct}%`;
  document.getElementById('score-ring').style.setProperty('--pct', `${pct}%`);
  const grid = document.getElementById('recap-grid');
  grid.innerHTML = '';
  consultLog.forEach(cl => {
    const col = cl.pct>=70?'#22c55e':cl.pct>=40?'#f59e0b':'#ef4444';
    const d = document.createElement('div');
    d.className = 'recap-item';
    d.style.borderColor = col;
    d.innerHTML = `<h4>${cl.owner.emoji} ${cl.owner.name}</h4><p>${cl.pct}% — ${cl.score>0?'+':''}${cl.score} pts<br><span style="font-size:12px;color:#64748b">${cl.owner.badgeText}</span></p>`;
    grid.appendChild(d);
  });
  const tips = document.getElementById('res-tips');
  tips.innerHTML = '';
  const globalTips = [
    {cls:'tip-good', text:'Valider l\'émotion AVANT de donner l\'information médicale (technique du pivot empathique)'},
    {cls:'tip-good', text:'Adapter son langage au profil du propriétaire en temps réel'},
    {cls:'tip-warn', text:'Éviter les formules creuses ("je comprends", "ne vous inquiétez pas") sans contenu'},
    {cls:'tip-warn', text:'Ne jamais minimiser sans avoir examiné l\'animal'},
    {cls:'tip-bad', text:'Jamais de confrontation directe avec un "sachant" — utiliser le "oui... et"'},
    {cls:'tip-bad', text:'Jamais de culpabilisation avec un propriétaire dans le déni'},
  ];
  globalTips.forEach(tip => {
    const li = document.createElement('li');
    const icons = {tip_good:'✅', tip_warn:'⚠️', tip_bad:'❌'};
    li.style.cssText = 'font-size:13px;color:#374151;margin:7px 0;line-height:1.6;list-style:none;display:flex;gap:8px;align-items:flex-start';
    const icn = tip.cls.replace('-','_');
    li.innerHTML = `<span>${icons[icn]||'•'}</span><span>${tip.text}</span>`;
    tips.appendChild(li);
  });
  document.getElementById('result-screen').style.display = 'flex';
}

function restartGame() {
  scene.children.length = 0;
  ownerMeshes.length = 0;
  document.getElementById('result-screen').style.display = 'none';
  document.getElementById('patients-done').innerHTML = '';
  const a = new THREE.AmbientLight(0xfff5e4,0.55); scene.add(a);
  const s = new THREE.DirectionalLight(0xfff8f0,0.9); s.position.set(6,12,8); s.castShadow=true; scene.add(s);
  const f = new THREE.DirectionalLight(0xe0f0ff,0.35); f.position.set(-6,4,-6); scene.add(f);
  buildRoom();
  OWNERS.forEach((o,i) => makeChar(o,ownerPositions[i][0],ownerPositions[i][2]));
  vetGroup = makeVet();
  playerX=0; playerZ=3; totalScore=0; consultedOwners=[]; consultLog=[]; stepScores=[]; currentStep=0;
  gameState='waiting'; updateHUD(); animate();
}

// ═══════════════════════════════════════════════════════════
//  INPUT
// ═══════════════════════════════════════════════════════════
document.addEventListener('keydown', e => {
  keys[e.key.toLowerCase()] = true;
  if((e.key==='e'||e.key==='E') && nearbyOwner && gameState==='waiting')
    openConsult(nearbyOwner.owner, nearbyOwner.index);
  if(e.key==='Escape') closeConsult();
  if(['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)) e.preventDefault();
});
document.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

const delay = ms => new Promise(r => setTimeout(r, ms));
</script>
</body>
</html>
