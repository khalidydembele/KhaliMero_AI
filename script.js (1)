// --- CONFIGURATION GROQ & IDENTITÉ ---
const GROQ_API_KEY = "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"; 

const MODELES_TEXTE = [
    "qwen/qwen3.8-27b",
    "openai/gpt-oss-120b",
    "openai/gpt-oss-20b",
    "qwen-2.5-32b-instruct"
];

let imageBase64 = "";
let modeActif = "normal";
let userLocation = null;
let styleImageActif = "photorealistic"; // Style par défaut

let conversations = [];
try {
    conversations = JSON.parse(localStorage.getItem('khalimero_chats')) || [];
} catch (e) {
    conversations = [];
    localStorage.removeItem('khalimero_chats');
}

let currentChatId = null;
let recognition = null;
let isListening = false;
const applicationsCodeStore = {};

// --- INITIALISATION ---
window.addEventListener('DOMContentLoaded', () => {
    setTimeout(() => {
        const splash = document.getElementById('splash-screen');
        if (splash) {
            splash.classList.add('fade-out');
            setTimeout(() => splash.style.display = 'none', 800);
        }
    }, 1800);

    const savedTheme = localStorage.getItem('khalimero_theme');
    if (savedTheme === 'light') {
        document.body.classList.add('light-theme');
        const icon = document.getElementById('theme-icon');
        if (icon) icon.textContent = 'light_mode';
    }

    // Gestion des clics sur les pills de style d'image
    document.querySelectorAll('.style-pill').forEach(pill => {
        pill.addEventListener('click', () => {
            document.querySelectorAll('.style-pill').forEach(p => p.classList.remove('active'));
            pill.classList.add('active');
            styleImageActif = pill.getAttribute('data-style');
        });
    });

    afficherListeHistorique();
    afficherEcranAccueil();
    initialiserReconnaissanceVocale();
});

// --- GÉOLOCALISATION INTELLIGENTE ET PERSISTANTE ---
async function activerGeolocalisation() {
    const geoBtn = document.getElementById('geo-btn');
    if (!navigator.geolocation) return alert("Géolocalisation non supportée.");

    document.getElementById('welcome-screen')?.remove();
    ajouterMessageBotHTML(`📍 <strong>Recherche de votre position précise...</strong> 🛰️`);

    navigator.geolocation.getCurrentPosition(
        async (position) => {
            const lat = position.coords.latitude;
            const lon = position.coords.longitude;
            
            try {
                const response = await fetch(`https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lon}`);
                const data = await response.json();
                
                userLocation = {
                    lat, lon,
                    adresse: data.display_name || "Adresse inconnue",
                    ville: data.address?.city || data.address?.town || data.address?.village || "Ville ou quartier inconnu",
                    pays: data.address?.country || "Pays inconnu"
                };

                const contentGeo = `
                    <div class="geo-card">
                        <div class="geo-card-header">
                            <span class="material-symbols-outlined">my_location</span>
                            <span>Position exacte détectée</span>
                        </div>
                        <p>📍 <strong>Lieu :</strong> ${userLocation.ville}, ${userLocation.pays}</p>
                        <p style="font-size: 0.8rem; color: var(--text-secondary); margin-top:4px;">${userLocation.adresse}</p>
                        <iframe class="geo-card-map" src="https://maps.google.com/maps?q=${lat},${lon}&z=15&output=embed"></iframe>
                    </div>`;

                ajouterMessageBotHTML(`📍 <strong>Géolocalisation réussie !</strong><br>Voici votre position actuelle :${contentGeo}<br>Je connais maintenant votre emplacement exact. Posez-moi des questions sur les environs, la météo ou les trajets ! 🚀`);
            } catch (error) {
                alert("Impossible de récupérer les détails de la position.");
            }
        },
        () => {
            alert("Accès à la position refusé ou indisponible.");
        },
        { enableHighAccuracy: true, timeout: 10000 }
    );
}

// --- GÉNÉRATEUR D'IMAGE AVANCÉ AVEC STYLES PROFESSIONNELS ---
function genererImageURL(prompt) {
    let cleanPrompt = prompt.toLowerCase().replace(/(génère|générer|dessine|crée une image)/gi, '').trim();
    if (!cleanPrompt) cleanPrompt = "artistic high quality concept";

    // Enrichissement professionnel du prompt selon le style choisi
    let styleModifiers = "";
    switch(styleImageActif) {
        case "photorealistic":
            styleModifiers = ", hyperrealistic, 8k resolution, cinematic lighting, photorealistic photography, sharp focus";
            break;
        case "cyberpunk":
            styleModifiers = ", cyberpunk style, neon lights, futuristic city, dark atmosphere, vibrant glowing colors, octane render";
            break;
        case "anime":
            styleModifiers = ", anime aesthetic, manga style, studio ghibli vibe, vibrant colors, detailed line art";
            break;
        case "oil":
            styleModifiers = ", classical oil painting, textured brushstrokes, masterpiece, artistic canvas style";
            break;
        case "3d":
            styleModifiers = ", 3D Pixar style, cute claymation, smooth surfaces, vibrant studio lighting, Unreal Engine 5 render";
            break;
        default:
            styleModifiers = ", high quality concept art";
    }

    const finalPrompt = cleanPrompt + styleModifiers;
    const url = `https://image.pollinations.ai/prompt/${encodeURIComponent(finalPrompt)}?width=1024&height=1024&nologo=true&seed=${Math.floor(Math.random() * 1000000)}`;

    return `
        <div class="gemini-img-card">
            <div class="gemini-img-wrapper" onclick="ouvrirModalImage('${url}')">
                <div class="img-skeleton"></div>
                <img src="${url}" alt="Génération KhaliMero" onload="this.classList.add('loaded'); this.previousElementSibling.style.display='none';">
                <div class="gemini-img-overlay">
                    <span class="material-symbols-outlined expand-icon">fullscreen</span>
                </div>
            </div>
            <div style="padding: 8px 12px; font-size: 0.75rem; color: var(--text-secondary); background: var(--bg-surface); border-top: 1px solid var(--border-color);">
                ✨ Style : <strong style="color: var(--accent-color);">${styleImageActif.toUpperCase()}</strong>
            </div>
        </div>
    `;
}

// --- FONCTIONNALITÉS UTILISATEUR ---
function viderChatActif() {
    if (!currentChatId) {
        afficherEcranAccueil();
        return;
    }
    if (confirm("Voulez-vous vider les messages de cette discussion en cours ?")) {
        const chat = conversations.find(c => c.id === currentChatId);
        if (chat) {
            chat.messages = [];
            sauvegarderLocal();
        }
        const container = document.getElementById('chat-container');
        if (container) container.innerHTML = "";
        afficherEcranAccueil();
    }
}

function mettreAJourCompteur(el) {
    const counter = document.getElementById('char-counter');
    if (counter) {
        const len = el.value.length;
        counter.textContent = `${len} / 2000`;
    }
}

function basculerTheme() {
    document.body.classList.toggle('light-theme');
    const isLight = document.body.classList.contains('light-theme');
    const icon = document.getElementById('theme-icon');
    
    if (icon) {
        icon.textContent = isLight ? 'light_mode' : 'dark_mode';
    }
    localStorage.setItem('khalimero_theme', isLight ? 'light' : 'dark');
}

// --- GESTION MODES (IMAGE & PROGRAMMEUR) ---
function activerModeProgrammeur() {
    modeActif = "programmeur";
    document.getElementById('image-style-bar').style.display = 'none';
    document.getElementById('welcome-screen')?.remove();
    ajouterMessageBotHTML(`⚡ <strong>Mode Programmeur Web Activé.</strong><br>Décrivez l'application web ou le composant à créer.`);
}

function activerModeImage() {
    modeActif = "image";
    // Afficher la barre de choix de style d'image
    document.getElementById('image-style-bar').style.display = 'flex';
    document.getElementById('welcome-screen')?.remove();
    ajouterMessageBotHTML(`🎨 <strong>Mode Génération d'Image Activé.</strong><br>Sélectionnez un style ci-dessous et décrivez l'image que vous désirez générer.`);
}

// --- GESTION MENU PIÈCE JOINTE (+) ---
function toggleAttachMenu(e) {
    e.stopPropagation();
    const dropdown = document.getElementById('attach-dropdown-menu');
    const triggerBtn = document.getElementById('attach-trigger-btn');
    
    dropdown?.classList.toggle('show');
    triggerBtn?.classList.toggle('open');
}

document.addEventListener('click', () => {
    const dropdown = document.getElementById('attach-dropdown-menu');
    const triggerBtn = document.getElementById('attach-trigger-btn');
    
    if (dropdown && dropdown.classList.contains('show')) {
        dropdown.classList.remove('show');
        triggerBtn.classList.remove('open');
    }
});

// --- RECONNAISSANCE VOCALE ---
function initialiserReconnaissanceVocale() {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (SpeechRecognition) {
        recognition = new SpeechRecognition();
        recognition.continuous = false;
        recognition.lang = 'fr-FR';
        recognition.interimResults = false;

        recognition.onstart = () => {
            isListening = true;
            document.getElementById('mic-btn')?.classList.add('listening');
        };

        recognition.onresult = (event) => {
            const transcript = event.results[0][0].transcript;
            const inputField = document.getElementById('user-input');
            if (inputField) {
                inputField.value = transcript;
                ajusterHauteur(inputField);
                mettreAJourCompteur(inputField);
            }
        };

        recognition.onend = () => {
            isListening = false;
            document.getElementById('mic-btn')?.classList.remove('listening');
            const inputField = document.getElementById('user-input');
            if (inputField && inputField.value.trim() !== '') {
                envoyer();
            }
        };

        recognition.onerror = () => {
            isListening = false;
            document.getElementById('mic-btn')?.classList.remove('listening');
        };
    } else {
        const micBtn = document.getElementById('mic-btn');
        if (micBtn) micBtn.style.display = 'none';
    }
}

function basculerDictéeVocale() {
    if (!recognition) return alert("Micro non supporté sur ce navigateur.");
    if (isListening) {
        recognition.stop();
    } else {
        recognition.start();
    }
}

// --- INTERFACE & NAVIGATION ---
function ajusterHauteur(el) {
    el.style.height = '24px';
    el.style.height = Math.min(el.scrollHeight, 100) + 'px';
}

function sauvegarderLocal() {
    try {
        localStorage.setItem('khalimero_chats', JSON.stringify(conversations));
    } catch (e) {
        conversations.forEach(c => {
            c.messages.forEach(m => {
                if (m.content && m.content.includes('data:image')) {
                    m.content = "[Image envoyée]";
                }
            });
        });
        try {
            localStorage.setItem('khalimero_chats', JSON.stringify(conversations));
        } catch (err) {
            console.warn("Stockage saturé malgré le nettoyage.");
        }
    }
}

function afficherListeHistorique() {
    const list = document.getElementById('history-list');
    if (!list) return;
    list.innerHTML = "";

    conversations.forEach(chat => {
        const item = document.createElement('div');
        item.className = `history-item ${chat.id === currentChatId ? 'active' : ''}`;
        item.innerHTML = `
            <span class="history-title" onclick="chargerChat(${chat.id})">💬 ${chat.titre || "Discussion"}</span>
            <span class="material-symbols-outlined delete-chat-btn" onclick="effacerUnChat(event, ${chat.id})">close</span>
        `;
        list.appendChild(item);
    });
}

function afficherEcranAccueil() {
    currentChatId = null;
    modeActif = "normal";
    document.getElementById('image-style-bar').style.display = 'none';
    afficherListeHistorique();

    const container = document.getElementById('chat-container');
    if (container) {
        container.innerHTML = `
            <div class="welcome-screen" id="welcome-screen">
                <h1 class="welcome-title">Bonjour, c'est KhaliMero ✨</h1>
                <p>Comment puis-je vous aider aujourd'hui ?</p>
                <div class="suggestions-grid">
                    <div class="suggestion-card" onclick="activerModeImage()">
                        <span>🎨 Générer une image</span>
                        <span class="material-symbols-outlined">image</span>
                    </div>
                    <div class="suggestion-card" onclick="activerModeProgrammeur()">
                        <span>💻 Mode Programmeur</span>
                        <span class="material-symbols-outlined">code</span>
                    </div>
                    <div class="suggestion-card" onclick="activerGeolocalisation()">
                        <span>📍 Me géolocaliser</span>
                        <span class="material-symbols-outlined">location_on</span>
                    </div>
                </div>
            </div>`;
    }
}

function nouveauChat() {
    currentChatId = Date.now();
    modeActif = "normal";
    conversations.unshift({ id: currentChatId, titre: "Discussion", messages: [] });
    sauvegarderLocal();
    afficherEcranAccueil();
}

function chargerChat(id) {
    currentChatId = id;
    modeActif = "normal";
    document.getElementById('image-style-bar').style.display = 'none';
    const chat = conversations.find(c => c.id === id);
    if (!chat) return;

    const container = document.getElementById('chat-container');
    container.innerHTML = "";

    if (chat.messages.length === 0) {
        afficherEcranAccueil();
    } else {
        chat.messages.forEach(msg => {
            if (msg.role === 'user') ajouterMessageUtilisateurHTML(msg.content);
            else ajouterMessageBotHTML(msg.content);
        });
        setTimeout(() => initialiserIframeApps(), 200);
    }
    afficherListeHistorique();
}

function effacerUnChat(e, id) {
    e.stopPropagation();
    conversations = conversations.filter(c => c.id !== id);
    sauvegarderLocal();
    if (currentChatId === id) afficherEcranAccueil();
    else afficherListeHistorique();
}

function effacerTout() {
    if (confirm("Supprimer tout l'historique ?")) {
        conversations = [];
        localStorage.removeItem('khalimero_chats');
        afficherEcranAccueil();
    }
}

function enregistrerMessage(role, content) {
    if (!currentChatId) {
        currentChatId = Date.now();
        conversations.unshift({ id: currentChatId, titre: "Discussion", messages: [] });
    }

    const chat = conversations.find(c => c.id === currentChatId);
    if (!chat) return;

    if (chat.messages.length === 0 && role === 'user') {
        let textePropre = content.replace(/<[^>]*>?/gm, '');
        chat.titre = textePropre.substring(0, 20) + (textePropre.length > 20 ? "..." : "");
    }

    chat.messages.push({ role, content });
    sauvegarderLocal();
    afficherListeHistorique();
}

function toggleSidebar() {
    document.getElementById('sidebar')?.classList.toggle('closed');
}

// --- GESTION IMAGES ET MODAL ---
function chargerImage(e) {
    const file = e.target.files[0];
    if (file) {
        const reader = new FileReader();
        reader.onload = function(evt) {
            imageBase64 = evt.target.result;
            const previewImg = document.getElementById('image-preview');
            const previewBox = document.getElementById('preview-box');
            if (previewImg) previewImg.src = imageBase64;
            if (previewBox) previewBox.style.display = 'block';
        };
        reader.readAsDataURL(file);
    }
}

function supprimerImage() {
    imageBase64 = "";
    const imgInput = document.getElementById('image-file-input');
    const camInput = document.getElementById('camera-file-input');
    const previewBox = document.getElementById('preview-box');
    
    if (imgInput) imgInput.value = "";
    if (camInput) camInput.value = "";
    if (previewBox) previewBox.style.display = 'none';
}

function ouvrirModalImage(src) {
    const modal = document.getElementById('image-modal');
    const imgTarget = document.getElementById('img-modal-target');
    const dlBtn = document.getElementById('download-modal-btn');

    if (imgTarget) imgTarget.src = src;
    if (dlBtn) dlBtn.href = src;

    if (modal) {
        modal.style.display = "flex";
        setTimeout(() => modal.classList.add('active'), 10);
    }
}

function fermerModalImage(event) {
    if (event.target.id === "image-modal" || event.target.classList.contains("viewer-content")) {
        fermerModalImageDirect();
    }
}

function fermerModalImageDirect() {
    const modal = document.getElementById('image-modal');
    if (modal) {
        modal.classList.remove('active');
        setTimeout(() => {
            modal.style.display = "none";
            const target = document.getElementById('img-modal-target');
            if (target) target.src = "";
        }, 300);
    }
}

// --- CONSTRUCTION DES MESSAGES GROQ (AVEC GÉOLOCALISATION PERSISTANTE) ---
function construireMessagesGroq(systemPrompt, userText, base64Img) {
    let messages = [{ role: "system", content: systemPrompt }];

    if (currentChatId) {
        const chat = conversations.find(c => c.id === currentChatId);
        if (chat && chat.messages) {
            chat.messages.forEach(m => {
                let cleanContent = typeof m.content === 'string' ? m.content.replace(/<[^>]*>?/gm, '').trim() : "";
                if (cleanContent && !cleanContent.includes("[Image envoyée]")) {
                    messages.push({
                        role: m.role === 'user' ? 'user' : 'assistant',
                        content: cleanContent
                    });
                }
            });
        }
    }

    if (base64Img) {
        messages.push({
            role: "user",
            content: [
                { type: "text", text: userText || "Décris et analyse cette image." },
                { type: "image_url", image_url: { url: base64Img } }
            ]
        });
    } else {
        messages.push({
            role: "user",
            content: userText || "Analyse."
        });
    }

    return messages;
}

// --- APPEL API GROQ ---
async function appelerApiGroq(systemPrompt, userText, base64Img) {
    let dernierErreur = "";
    const messagesPayload = construireMessagesGroq(systemPrompt, userText, base64Img);

    for (const model of MODELES_TEXTE) {
        try {
            const res = await fetch("https://api.groq.com/openai/v1/chat/completions", {
                method: "POST",
                headers: {
                    "Authorization": `Bearer ${GROQ_API_KEY.trim()}`,
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({
                    model: model,
                    messages: messagesPayload,
                    temperature: 0.6,
                    max_completion_tokens: 2048,
                    top_p: 0.95
                })
            });

            if (res.ok) {
                const data = await res.json();
                let texteReponse = data.choices?.[0]?.message?.content;
                if (texteReponse) return texteReponse;
            } else {
                const errData = await res.json();
                dernierErreur = errData?.error?.message || res.statusText;
            }
        } catch (e) {
            dernierErreur = e.message;
        }
    }
    throw new Error(dernierErreur || "Erreur de communication avec l'API Groq.");
}

// --- FONCTION PRINCIPALE D'ENVOI ---
async function envoyer() {
    const inputField = document.getElementById('user-input');
    const texte = inputField.value.trim();
    const currentImg = imageBase64;

    if (!texte && !currentImg) return;

    document.getElementById('welcome-screen')?.remove();

    let htmlUser = texte;
    if (currentImg) {
        htmlUser = `<img src="${currentImg}" style="max-height:150px; display:block; margin-bottom:8px; border-radius:8px;">` + (texte || "Analyse de cette photo.");
    }

    ajouterMessageUtilisateurHTML(htmlUser);
    enregistrerMessage('user', htmlUser);

    inputField.value = "";
    inputField.style.height = '24px';
    mettreAJourCompteur(inputField);
    supprimerImage();

    const botRow = ajouterLigneBotVide();
    botRow.innerHTML = `
        <div class="gemini-thinking">
            <div class="magic-pulse-orb"></div>
            <span>${currentImg ? "KhaliMero observe votre image... 👁️" : "KhaliMero réfléchit... ✨"}</span>
        </div>`;

    try {
        let reponseFinale = "";
        const estDemandeImage = modeActif === "image" || /(génère|générer|dessine|crée une image)/i.test(texte);
        const estDemandeCode = modeActif === "programmeur" || /(crée|créer|calculatrice|application|app|jeu|site|code|html)/i.test(texte);

        let systemPrompt = `Tu es KhaliMero AI, une intelligence artificielle ultra-avancée, élégante et performante. 
IDENTITÉ : Conçu et développé par Khalidy Souleymane Dembélé et Diakary Dja Camara, brillants étudiants en ingénierie informatique à l'université INTECsup. 
REPONDS DIRECTEMENT ET DE MANIERE CLAIRE ET CAPTIVANTE.`;

        // Injection permanente de la géolocalisation dans l'IA si elle est connue
        if (userLocation) {
            systemPrompt += ` L'utilisateur se trouve actuellement à : ${userLocation.ville}, ${userLocation.pays} (Latitude: ${userLocation.lat}, Longitude: ${userLocation.lon}, Adresse: ${userLocation.adresse}). Tiens compte de cet emplacement géographique précis pour toutes ses requêtes contextuelles.`;
        }

        if (estDemandeImage && !currentImg) {
            await new Promise(r => setTimeout(r, 600));
            reponseFinale = genererImageURL(texte);
        } else {
            if (estDemandeCode) {
                systemPrompt = "Tu es KhaliMero, un expert développeur web de génie. Fournis le code HTML/CSS/JS complet dans un seul bloc markdown ```html ... ```. Ne mets aucun texte explicatif avant ou après le bloc.";
            }
            const rawText = await appelerApiGroq(systemPrompt, texte, currentImg);
            reponseFinale = formatterReponse(rawText);
        }

        botRow.innerHTML = `
            <div class="message-content">${reponseFinale}</div>
            <div class="bot-footer">
                <button class="copy-msg-btn" onclick="copierTexteReponse(this)">
                    <span class="material-symbols-outlined">content_copy</span>
                    <span>Copier</span>
                </button>
                <span class="bot-disclaimer">KhaliMero peut se tromper. Vérifiez les informations.</span>
            </div>`;

        enregistrerMessage('bot', reponseFinale);
        
        setTimeout(() => initialiserIframeApps(), 150);

        const container = document.getElementById('chat-container');
        if(container) container.scrollTop = container.scrollHeight;

    } catch (err) {
        botRow.innerHTML = `<div class="message-content" style="color:#ff8b8b;">⚠️ Erreur : ${err.message}</div>`;
    }
}

// --- FORMATAGE DES APPLICATIONS WEB ---
function formatterReponse(texte) {
    let textePropre = texte
        .replace(/<think>[\s\S]*?<\/think>/gi, '') 
        .replace(/All constraints satisfied\.?[\s\S]*?\n/gi, '') 
        .trim();

    const codeBlocks = [];
    const regexBlock = /```(?:html|xml|web|javascript|js)?[\s\t]*\r?\n([\s\S]*?)(?:```|$)/gi;

    let textWithPlaceholders = textePropre.replace(regexBlock, function(match, code) {
        const id = codeBlocks.length;
        codeBlocks.push(code.trim());
        return `___APP_BLOCK_${id}___`;
    });

    if (codeBlocks.length > 0) {
        let resultHtml = textWithPlaceholders
            .replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>")
            .replace(/\*(.*?)\*/g, "<em>$1</em>")
            .replace(/\n/g, "<br>");
        
        codeBlocks.forEach((codeBrut, index) => {
            const cardId = `app-card-${Date.now()}-${index}`;
            applicationsCodeStore[cardId] = codeBrut;

            const codeEchappeHTML = codeBrut.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");

            const appCardHtml = `
                <div class="app-card-wrapper" id="${cardId}">
                    <div class="app-card-header">
                        <div class="app-title-tag">
                            <span class="material-symbols-outlined">devices</span> Application Web
                        </div>
                        <div class="app-actions">
                            <button class="app-action-icon-btn" title="Plein écran" onclick="basculerPleinEcran('${cardId}')">
                                <span class="material-symbols-outlined">fullscreen</span>
                            </button>
                            <button class="app-action-icon-btn" title="Voir le code source" onclick="basculerTiroirCode('${cardId}')">
                                <span class="material-symbols-outlined">code</span>
                            </button>
                        </div>
                    </div>
                    <div class="app-viewport">
                        <iframe sandbox="allow-scripts allow-modals allow-forms allow-same-origin"></iframe>
                    </div>
                    <div class="code-collapsible-drawer" id="drawer-${cardId}">
                        <div class="code-collapsible-header">
                            <span>Code source complet</span>
                            <button class="copy-inner-btn" onclick="copierCodeSource('${cardId}')">
                                <span class="material-symbols-outlined" style="font-size:14px;">content_copy</span> Copier
                            </button>
                        </div>
                        <pre><code>${codeEchappeHTML}</code></pre>
                    </div>
                </div>`;

            resultHtml = resultHtml.replace(`___APP_BLOCK_${index}___`, appCardHtml);
        });
        return resultHtml;
    }

    return textePropre
        .replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>")
        .replace(/\*(.*?)\*/g, "<em>$1</em>")
        .replace(/\n/g, "<br>");
}

// --- GESTION DES IFRAMES ---
function initialiserIframeApps() {
    document.querySelectorAll('.app-card-wrapper').forEach(card => {
        const cardId = card.id;
        const iframe = card.querySelector('iframe');
        const codeBrut = applicationsCodeStore[cardId];

        if (iframe && codeBrut) {
            try {
                const doc = iframe.contentDocument || iframe.contentWindow.document;
                doc.open();
                doc.write(codeBrut);
                doc.close();
            } catch (e) {
                console.error("Erreur iframe:", e);
            }
        }
    });
}

function basculerPleinEcran(cardId) {
    const card = document.getElementById(cardId);
    if (card) {
        card.classList.toggle('fullscreen-mode');
        const icon = card.querySelector('[onclick*="basculerPleinEcran"] span');
        if (icon) {
            icon.textContent = card.classList.contains('fullscreen-mode') ? 'fullscreen_exit' : 'fullscreen';
        }
    }
}

function basculerTiroirCode(cardId) {
    const drawer = document.getElementById(`drawer-${cardId}`);
    if (drawer) {
        drawer.classList.toggle('open');
    }
}

function copierCodeSource(cardId) {
    const code = applicationsCodeStore[cardId];
    if (code) {
        navigator.clipboard.writeText(code);
        alert("Code source copié dans le presse-papier ! 📋");
    }
}

function copierTexteReponse(btn) {
    const messageRow = btn.closest('.message-row');
    const content = messageRow?.querySelector('.message-content');
    if (content) {
        const textToCopy = content.innerText;
        navigator.clipboard.writeText(textToCopy);
        const originalText = btn.innerHTML;
        btn.innerHTML = `<span class="material-symbols-outlined">check</span><span>Copié !</span>`;
        setTimeout(() => { btn.innerHTML = originalText; }, 2000);
    }
}

function ajouterMessageUtilisateurHTML(htmlContent) {
    const container = document.getElementById('chat-container');
    if (!container) return;

    const row = document.createElement('div');
    row.className = 'message-row user';
    row.innerHTML = `<div class="message-content">${htmlContent}</div>`;
    container.appendChild(row);
    container.scrollTop = container.scrollHeight;
}

function ajouterMessageBotHTML(htmlContent) {
    const container = document.getElementById('chat-container');
    if (!container) return;

    const row = document.createElement('div');
    row.className = 'message-row bot';
    row.innerHTML = `
        <div class="message-content">${htmlContent}</div>
        <div class="bot-footer">
            <button class="copy-msg-btn" onclick="copierTexteReponse(this)">
                <span class="material-symbols-outlined">content_copy</span>
                <span>Copier</span>
            </button>
            <span class="bot-disclaimer">KhaliMero peut se tromper. Vérifiez les informations.</span>
        </div>`;
    container.appendChild(row);
    container.scrollTop = container.scrollHeight;
}

function ajouterLigneBotVide() {
    const container = document.getElementById('chat-container');
    if (!container) return null;

    const row = document.createElement('div');
    row.className = 'message-row bot';
    container.appendChild(row);
    container.scrollTop = container.scrollHeight;
    return row;
}

document.addEventListener('DOMContentLoaded', () => {
    const inputField = document.getElementById('user-input');
    if (inputField) {
        inputField.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                envoyer();
            }
        });
    }
});
