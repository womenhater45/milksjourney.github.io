<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Milkman's Fediverse Adventure</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'VT323', monospace;
            background-color: #008080;
            color: #c0c0c0;
        }

        .window {
            border: 2px solid #c0c0c0;
            border-right-color: #000000;
            border-bottom-color: #000000;
            background-color: #c0c0c0;
            box-shadow: 2px 2px 0px #000000;
        }

        .title-bar {
            background: linear-gradient(to right, #000080, #1084d0);
            color: white;
            padding: 2px 4px;
            font-weight: bold;
        }

        .buddy-list-container {
            max-height: 250px;
            overflow-y: auto;
            border: 1px inset #808080;
            padding: 4px;
        }

        .buddy-list-item, .choice-button {
            border: 1px solid #c0c0c0;
            border-right-color: #000000;
            border-bottom-color: #000000;
            background-color: #c0c0c0;
            cursor: pointer;
        }

            .buddy-list-item.flash {
                animation: flash-buddy 1.5s infinite;
            }

        @keyframes flash-buddy {
            0%, 100% {
                background-color: #c0c0c0;
            }

            50% {
                background-color: #ffff00;
            }
        }

        .choice-button:hover {
            background-color: #a0a0a0;
        }

        .buddy-list-item.offline {
            color: #808080;
            cursor: not-allowed;
        }

        .buddy-list-item.online {
            color: #000000;
            font-weight: bold;
        }

        .chat-window, .timeline-window, .diary-window {
            background-color: white;
            color: black;
            height: 300px;
            overflow-y: auto;
            padding: 8px;
            border: 2px inset #808080;
        }

        .post, .diary-entry {
            border-bottom: 1px solid #e0e0e0;
            padding-bottom: 8px;
            margin-bottom: 8px;
        }

        .choices-container {
            padding: 8px;
            border-top: 2px solid #c0c0c0;
        }

        .status-bar {
            border-top: 1px solid #808080;
            border-bottom: 1px solid white;
            padding: 2px 4px;
            font-size: 0.9em;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-7xl mx-auto grid grid-cols-1 lg:grid-cols-4 gap-4">
        <!-- Left Column: Buddy List, Profile -->
        <div class="lg:col-span-1 space-y-4">
            <div class="window h-fit">
                <div class="title-bar"><span>Mainframe</span></div>
                <div class="p-2 bg-[#008080]">
                    <div class="p-2 window">
                        <div class="p-2 bg-white text-black mb-2 border-b-2 border-gray-400">
                            <div class="font-bold text-lg">Milkman</div>
                            <div id="follower-count" class="text-sm">Followers: 0</div>
                            <div id="dinar-count" class="text-sm">Dinars: 0</div>
                        </div>
                        <div class="font-bold text-black">▼ Buddy List</div>
                        <div id="buddy-list" class="buddy-list-container pl-4 space-y-1 mt-1"></div>
                    </div>
                </div>
            </div>
            <div class="window h-fit">
                <div class="title-bar"><span>Profile</span></div>
                <div class="p-4 bg-white text-black" id="profile-window">
                    <p class="text-gray-500">Select a buddy to view their profile.</p>
                </div>
            </div>
        </div>

        <!-- Middle Column: Chat, Diary -->
        <div class="lg:col-span-2 space-y-4">
            <div class="window">
                <div class="title-bar"><span id="chat-window-title">FediMessenger</span></div>
                <div class="chat-window" id="chat-window" style="height: 450px;">
                    <p class="text-gray-500">Connecting to the Fediverse... Click a flashing name in the Buddy List to begin.</p>
                </div>
                <div class="choices-container" id="choices-container"></div>
                <div class="status-bar"><span id="status-text">Idle.</span></div>
            </div>
            <div class="window">
                <div class="title-bar"><span>Milkman's Diary</span></div>
                <div class="diary-window" id="diary-window" style="height: 200px;">
                    <p class="text-gray-500">My thoughts...</p>
                </div>
            </div>
        </div>

        <!-- Right Column: Timeline -->
        <div class="lg:col-span-1 space-y-4">
            <div class="window">
                <div class="title-bar"><span>Fediverse Timeline</span></div>
                <div class="timeline-window" id="timeline-window" style="height: 700px;">
                    <p class="text-gray-500">Public posts will appear here...</p>
                </div>
            </div>
        </div>
    </div>

    <div id="treasure-modal" class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center hidden z-50">
        <div class="window p-4 max-w-sm text-black">
            <div class="title-bar"><span>!!! TREASURE FOUND !!!</span></div>
            <div class="p-4 bg-white">
                <p class="text-lg mb-4">While digging through an archived instance, you found a file: `old_wallet.dat`. The password was `password123`.</p>
                <p class="text-xl font-bold text-green-600 mb-4">Inside, you find 1.5 long-forgotten FediCoin, valued at 40,000 Dinars!</p>
                <p class="mb-4">This could change everything...</p>
                <button onclick="document.getElementById('treasure-modal').classList.add('hidden')" class="choice-button w-full p-2">Awesome!</button>
            </div>
        </div>
    </div>

    <div id="debug-modal" class="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center hidden z-50">
        <div class="window p-4 max-w-md text-black">
            <div class="title-bar"><span>DEBUG: JUMP TO ARC 2</span></div>
            <div class="p-4 bg-white space-y-2">
                <p class="text-lg mb-2">Select your Arc 1 ending to begin Arc 2:</p>
                <button onclick="jumpToArc2('power')" class="choice-button w-full p-2">Sided with Teto (Power)</button>
                <button onclick="jumpToArc2('integrity')" class="choice-button w-full p-2">Sided with Ahab (Integrity)</button>
                <button onclick="jumpToArc2('ghost')" class="choice-button w-full p-2">Sided with Judge Dread (Ghost)</button>
                <button onclick="jumpToArc2('chaos')" class="choice-button w-full p-2">Sided with ZeroCool (Chaos)</button>
                <hr class="my-2">
                <button onclick="document.getElementById('debug-modal').classList.add('hidden')" class="choice-button w-full p-2 bg-red-300">Cancel</button>
            </div>
        </div>
    </div>

    <script>
        // DOM Elements
        const followerCountEl = document.getElementById('follower-count');
        const dinarCountEl = document.getElementById('dinar-count');
        const timelineWindow = document.getElementById('timeline-window');
        const buddyListContainer = document.getElementById('buddy-list');
        const treasureModal = document.getElementById('treasure-modal');
        const chatWindowTitle = document.getElementById('chat-window-title');
        const chatWindow = document.getElementById('chat-window');
        const choicesContainer = document.getElementById('choices-container');
        const statusText = document.getElementById('status-text');
        const diaryWindow = document.getElementById('diary-window');
        const debugModal = document.getElementById('debug-modal');
        const profileWindow = document.getElementById('profile-window');

        // Game State
        let gameState = {
            currentConversationKey: null,
            buddies: {
                admin: { name: 'InstanceAdmin', isVisible: true, followers: 1200, bio: 'Just trying to keep the lights on and the bits flowing. He/Him.', online: true, hasNewMessage: true, currentNode: 'start', chatHistory: [], completedNodes: new Set() },
                artfriend: { name: 'WholesomeArtist', isVisible: false, followers: 850, bio: 'Pixel artist and lover of all things cozy. Let\'s make the world a kinder place!', online: false, hasNewMessage: false, currentNode: 'artfriend_convo_start', chatHistory: [], completedNodes: new Set() },
                rival: { name: 'BoostMasterFlex', isVisible: false, followers: 5600, bio: 'I post facts and you post cringe. That\'s just how it is.', online: false, hasNewMessage: false, currentNode: 'rival_convo_start', chatHistory: [], completedNodes: new Set() },
                theorytoe: { name: 'TheoryToe', isVisible: false, followers: 150, bio: 'Epistemologist. Digital Cartographer. My theses are peer-reviewed by myself and found to be flawless.', online: false, hasNewMessage: false, currentNode: 'theorytoe_convo_start', chatHistory: [], completedNodes: new Set() },
                judgedread: { name: 'JudgeDread', isVisible: false, followers: 9800, bio: 'Old man trying to save the West with Star Wars posts. The Force has a liberal bias.', online: false, hasNewMessage: false, currentNode: 'judgedread_convo_start', chatHistory: [], completedNodes: new Set() },
                ahab: { name: 'CaptainAhab', isVisible: false, followers: 450, bio: 'From hell\'s heart I stab at thee; for hate\'s sake I spit my last breath at thee.', online: false, hasNewMessage: false, currentNode: 'ahab_convo_start', chatHistory: [], completedNodes: new Set() },
                owl: { name: 'Owl', isVisible: false, followers: 120, bio: 'Mortuary Technician. We all end up on the slab eventually.', online: false, hasNewMessage: false, currentNode: 'owl_convo_start', chatHistory: [], completedNodes: new Set() },
                sysop: { name: 'OldManBBS', isVisible: false, followers: 25, bio: 'NO CARRIER', online: false, hasNewMessage: false, currentNode: 'sysop_convo_start', chatHistory: [], completedNodes: new Set() },
                glitch: { name: 'Glitch', isVisible: false, followers: 5, bio: 'They took everything.', online: false, hasNewMessage: false, currentNode: 'glitch_convo_start', chatHistory: [], completedNodes: new Set() },
                chadcorp: { name: 'ChadCorp', isVisible: false, followers: 150000, bio: 'VP of Synergistic Innovation at OmniCorp. Let\'s leverage our assets. #Business', online: false, hasNewMessage: false, currentNode: 'chadcorp_convo_start', chatHistory: [], completedNodes: new Set() },
                zerocool: { name: 'ZeroCool', isVisible: false, followers: 0, bio: 'Mess with the best, die like the rest.', online: false, hasNewMessage: false, currentNode: 'zerocool_convo_start', chatHistory: [], completedNodes: new Set() },
                teto: { name: 'KasaneTeto', isVisible: false, followers: 999999, bio: 'CEO of UTAU.Host. I am the signal.', online: false, hasNewMessage: false, currentNode: 'teto_intro', chatHistory: [], completedNodes: new Set() },
                dinardealer: { name: 'DinarDealer', isVisible: false, followers: 1, bio: 'Discreet services for discerning clients.', online: false, hasNewMessage: false, currentNode: 'dinardealer_intro', chatHistory: [], completedNodes: new Set() },
                nierenstein: { name: 'Nierenstein', isVisible: false, followers: 25000, bio: 'Spectrobes is not a game, it is a way of life. Disney must be forced to make another.', online: false, hasNewMessage: false, currentNode: 'nierenstein_intro', chatHistory: [], completedNodes: new Set() },
                gumshoe: { name: 'DetectiveGumshoe', isVisible: false, followers: 3, bio: 'Private Eye. Cases solved for cheap. Dinars up front.', online: false, hasNewMessage: false, currentNode: 'gumshoe_intro', chatHistory: [], completedNodes: new Set() },
                spectrefan: { name: 'SpectrobeFanatic117', isVisible: false, followers: 12000, bio: 'Nierenstein is a FAKE FAN and I can prove it. #RealSpectrobesFan', online: false, hasNewMessage: false, currentNode: 'spectrefan_intro', chatHistory: [], completedNodes: new Set() }
            },
            milkman: {
                influence: 0,
                hasTreasure: false,
                integrity: 10,
                followers: 0,
                dinars: 0,
            },
            plot: {
                knowsAdminIsTetoAlly: false,
                knowsAhabsTrueQuest: false,
                theoryToeBetrayed: false,
                arc1_ending: null,
                nierenstein_murdered: false,
                knowsNierensteinFaked: false,
            }
        };

        const randomPosts = [
            { user: 'CatLover42', text: 'Just saw a cat chase a laser pointer for 10 minutes straight. My life is complete. #caturday' },
            { user: 'BreadEnthusiast', text: 'PRO TIP: If you toast one side of the bread longer, you get a perfect crunchy/soft ratio. You\'re welcome.' },
            { user: 'ConspiracyGary', text: 'The squirrels are replacing the batteries in the pigeons again. I saw them. Don\'t believe the lies.' },
            { user: 'HaikuBot', text: 'Digital words flow,\nA story starts to unfold,\nClick the next choice now.' },
            { user: 'FernDad', text: 'My fern, Bartholomew, has a new leaf today. I am so proud of him.' },
            { user: 'ServerHamster', text: '*runs faster on wheel* squeak squeak! #serverlife' },
            { user: 'FimbulHater99', text: 'The price of a pack of smokes went up another 5 dinars. Thanks, FIMBUL. Hope that giant cigar of yours goes up like the Hindenburg. #FimbulTax' },
            { user: 'RoyalHerald', text: 'His Majesty King Fimbul has acquired the entire tobacco yield of the southern continent. The Great Ascension Cigar is now 15% complete! Rejoice!' }
        ];

        const story = {
            // --- ARC 1 ---
            'start': {
                diaryEntry: "Another day, another dinar. Cigarettes went up *again*. Thanks, Fimbul. Hope you choke on that giant cigar of yours.",
                dialogue: [{ speaker: 'InstanceAdmin', text: 'Welcome to Social.Fedi, Milkman! Glad to have you here.' }],
                choices: [{ text: '"How do I get popular?"', nextNode: 'admin_fame' }]
            },
            'admin_fame': {
                dialogue: [{ speaker: 'InstanceAdmin', text: 'Haha, straight to it. Well, you gotta post good content. Get boosts. Interact.' }],
                onChoice: (state) => { state.milkman.followers += 10; },
                nextNode: 'admin_end_intro'
            },
            'admin_end_intro': {
                timelinePost: { character: 'admin', text: 'Server load is spiking... probably just growing pains! #fediverse' },
                dialogue: [{ speaker: 'InstanceAdmin', text: 'Anyway, I gotta go fix a federation issue. Have fun out there!' }],
                onComplete: () => queueMessage('artfriend', 1000)
            },
            'artfriend_convo_start': {
                dialogue: [{ speaker: 'WholesomeArtist', text: 'Hey! I saw your first posts. Welcome! :)' }],
                choices: [{ text: '"Thanks! Your art is cool."', nextNode: 'artfriend_compliment' }]
            },
            'artfriend_compliment': {
                dialogue: [{ speaker: 'WholesomeArtist', text: 'Aw, thank you! That means a lot. Talk to you later!' }],
                onChoice: (state) => { state.milkman.followers += 15; },
                nextNode: 'artfriend_end'
            },
            'artfriend_end': {
                dialogue: [],
                onComplete: () => queueMessage('rival', 1000)
            },
            'rival_convo_start': {
                timelinePost: { character: 'rival', text: 'another day, another n00b thinking this is myspace. smh.' },
                dialogue: [{ speaker: 'BoostMasterFlex', text: 'lol, another newbie. saw your first post. cringe.' }],
                choices: [{ text: '"Whatever, dude."', nextNode: 'rival_whatever' }]
            },
            'rival_whatever': {
                dialogue: [{ speaker: 'BoostMasterFlex', text: 'heh. got some spine. maybe you won\'t get eaten alive. later, n00b.' }],
                onChoice: (state) => { state.milkman.followers += 20; },
                nextNode: 'rival_end'
            },
            'rival_end': {
                diaryEntry: "This BoostMasterFlex punk... thinks he's so tough. At least I can still afford a pack of smokes. For now.",
                timelinePost: { character: 'rival', text: 'owning n00bs is my brand and business is good' },
                dialogue: [],
                onComplete: () => queueMessage('theorytoe', 1000)
            },
            'theorytoe_convo_start': {
                dialogue: [
                    { speaker: 'TheoryToe', text: 'Salutations, nascent entity. I have algorithmically detected your presence in this sector of the datasphere.' },
                    { speaker: 'TheoryToe', text: 'Your interaction patterns are... sub-optimal. But they possess a certain chaotic potentiality. Fascinating.' }
                ],
                choices: [{ text: '"...who are you?"', nextNode: 'theorytoe_awkward' }]
            },
            'theorytoe_awkward': {
                dialogue: [{ speaker: 'TheoryToe', text: 'I am TheoryToe, a freelance epistemologist and digital cartographer. You may refer to me as Sensei. I am offering you, pro bono for now, my intellectual guidance.' }],
                onChoice: (state) => { state.milkman.followers = Math.max(0, state.milkman.followers - 5); },
                nextNode: 'theorytoe_end'
            },
            'theorytoe_end': {
                dialogue: [],
                onComplete: () => queueMessage('judgedread', 1000)
            },
            'judgedread_convo_start': {
                timelinePost: { character: 'judgedread', text: 'The Fediverse is like the Star Wars prequels. Full of good ideas, but bogged down by bad actors and trade disputes. We need more Jedi, not more senators.' },
                dialogue: [{ speaker: 'JudgeDread', text: 'Young man. I see you navigating this wretched hive of scum and villainy. Remember this: A lightsaber is a tool of defense, not aggression. Build, don\'t destroy.' }],
                choices: [{ text: 'Thank you for the wisdom.', nextNode: 'judgedread_end' }]
            },
            'judgedread_end': {
                dialogue: [{ speaker: 'JudgeDread', text: 'May the Force be with you.' }],
                onChoice: (state) => { state.milkman.integrity += 5; state.milkman.followers += 30; },
                nextNode: 'judgedread_end_2'
            },
            'judgedread_end_2': {
                dialogue: [],
                onComplete: () => queueMessage('ahab', 1000)
            },
            'ahab_convo_start': {
                dialogue: [{ speaker: 'CaptainAhab', text: 'I hunt a greater beast. The White Whale... the lost instance of TempusFugit.' }],
                choices: [{ text: '"A lost instance?"', nextNode: 'ahab_ask_instance' }]
            },
            'ahab_ask_instance': {
                dialogue: [{ speaker: 'CaptainAhab', text: 'Aye. The first. It went dark years ago. Here... an old archive mirror. `archive.tempusfugit.net/` See for yourself.' }],
                nextNode: 'ahab_found_treasure'
            },
            'ahab_found_treasure': {
                diaryEntry: "40,000 dinars! Holy... I can buy cigarettes for a year! Maybe even two! Take THAT, Fimbul!",
                dialogue: [
                    { speaker: 'Milkman', text: '...I looked through it. Found an old wallet file. It... it had 40,000 dinars worth of FediCoin in it.' },
                    { speaker: 'CaptainAhab', text: 'The sea provides! But coin is just dross, lad. The real treasure is the chase!' },
                ],
                onChoice: (state) => {
                    state.milkman.hasTreasure = true;
                    state.milkman.dinars = 40000;
                    state.milkman.followers += 50;
                    treasureModal.classList.remove('hidden');
                },
                nextNode: 'ahab_end'
            },
            'ahab_end': {
                dialogue: [],
                onComplete: () => queueMessage('dinardealer', 2000)
            },
            'dinardealer_intro': {
                dialogue: [
                    { speaker: 'DinarDealer', text: 'Psst. Hey. Heard you came into some money. Good for you.' },
                    { speaker: 'DinarDealer', text: 'I can make that money grow. I got clients. They need things done. A rumor spread, a rival\'s reputation tarnished... easy work for a man with your reach. You get paid, no questions asked.' }
                ],
                choices: [{ text: 'What kind of work?', nextNode: 'dinardealer_offer' }, { text: 'Get lost.', nextNode: 'dinardealer_end' }]
            },
            'dinardealer_offer': {
                dialogue: [{ speaker: 'DinarDealer', text: 'I got a client who wants to hurt ChadCorp\'s stock. Spread a rumor that their new crypto-synergy platform is a total scam. Easy 15,000 dinars.' }],
                choices: [{ text: 'I\'ll do it.', nextNode: 'dinardealer_accept' }, { text: 'Not a chance.', nextNode: 'dinardealer_end' }]
            },
            'dinardealer_accept': {
                timelinePost: { character: 'milkman', text: 'Hearing some sketchy things about OmniCorp\'s new crypto platform. Might be vaporware. Be careful out there, folks. #ChadCorp #OmniCorp #Scam' },
                onChoice: state => { state.milkman.dinars += 15000; state.milkman.integrity -= 10; },
                dialogue: [{ speaker: 'DinarDealer', text: 'Pleasure doing business with you. The dinars are in your account.' }],
                nextNode: 'dinardealer_end'
            },
            'dinardealer_end': {
                dialogue: [{ speaker: 'DinarDealer', text: 'My DMs are always open.' }],
                onComplete: () => queueMessage('chadcorp', 1500)
            },
            'chadcorp_convo_start': {
                timelinePost: { character: 'chadcorp', text: 'Just synergized some bleeding-edge paradigms with our Q3 growth strategy. Big things coming from OmniCorp! #Business #Innovation #Crypto' },
                dialogue: [
                    { speaker: 'ChadCorp', text: 'Milkman! Chad from OmniCorp here. We\'ve been watching your trajectory. Impressive stuff.' },
                    { speaker: 'ChadCorp', text: 'Heard you came into some capital. We\'re always looking for disruptive players to join our portfolio. Let\'s touch base and circle back on some win-win opportunities.' },
                ],
                choices: [{ text: 'No thanks, I\'m good.', nextNode: 'chadcorp_end' }]
            },
            'chadcorp_end': {
                dialogue: [{ speaker: 'ChadCorp', text: 'Alright, keep my v-card on file. Let\'s blue-sky this later.' }],
                onComplete: () => queueMessage('owl', 1000)
            },
            'owl_convo_start': {
                timelinePost: { character: 'owl', text: 'Another day, another body. The silence of the morgue is preferable to the noise online. Some people just take up too much space.' },
                dialogue: [{ speaker: 'Owl', text: 'You\'re Milkman.' }, { speaker: 'Owl', text: 'I\'ve been watching you. You remind me of someone I used to know. Someone who got too big for their boots.' }, { speaker: 'Owl', text: 'It didn\'t end well for them.' }],
                choices: [{ text: 'Is that a threat?', nextNode: 'owl_threat' }, { text: 'I don\'t know you.', nextNode: 'owl_dismiss' }]
            },
            'owl_threat': {
                dialogue: [{ speaker: 'Owl', text: 'It\'s an observation. The mortuary is very patient. It always gets its due.' }],
                onChoice: (state) => { state.milkman.influence -= 2; },
                nextNode: 'owl_end'
            },
            'owl_dismiss': {
                dialogue: [{ speaker: 'Owl', text: 'You will.' }],
                nextNode: 'owl_end'
            },
            'owl_end': {
                diaryEntry: "This Owl creep is getting on my nerves. I need a smoke. Can't believe a pack costs 50 dinars now. Fimbul's 'ascension' better be a damn firework show for that price.",
                timelinePost: { character: 'owl', text: 'Formaldehyde has a sharp, clean smell. It clears the air of impurities.' },
                dialogue: [],
                onComplete: () => queueMessage('sysop', 1000)
            },
            'sysop_convo_start': {
                dialogue: [{ speaker: 'OldManBBS', text: '+++ ATH0' }, { speaker: 'OldManBBS', text: 'You found the archives. TempusFugit wasn\'t an "instance". It was a protocol. A promise of a free network.' }],
                choices: [{ text: '"What happened to it?"', nextNode: 'sysop_explain' }]
            },
            'sysop_explain': {
                dialogue: [{ speaker: 'OldManBBS', text: 'It was stolen. Corrupted by a young prodigy who saw it not as a gift, but as a weapon. She built a backdoor into its heart. Her name was Teto.' }],
                nextNode: 'sysop_end'
            },
            'sysop_end': {
                dialogue: [{ speaker: 'OldManBBS', text: 'Be careful who you trust. NO CARRIER' }],
                onComplete: () => queueMessage('glitch', 1000)
            },
            'glitch_convo_start': {
                timelinePost: { character: 'glitch', text: 'Another federation error. Same one as last time. It\'s not a bug.' },
                dialogue: [{ speaker: 'Glitch', text: 'You\'re talking to SysOp. And Teto.' }, { speaker: 'Glitch', text: 'Don\'t be a fool. I ran an instance once. `cyberspace.zone`. Teto offered me power. Then she used her backdoor and took everything. My users, my data... everything.' }],
                choices: [{ text: 'Why are you telling me this?', nextNode: 'glitch_end' }]
            },
            'glitch_end': {
                dialogue: [{ speaker: 'Glitch', text: 'Because nobody else will. Don\'t end up like me.' }],
                onComplete: () => queueMessage('teto', 1000)
            },
            'teto_intro': {
                timelinePost: { character: 'teto', text: 'Amateurs play for followers. Professionals play for control. The board is set.' },
                dialogue: [{ speaker: 'KasaneTeto', text: 'Milkman. I have a proposition.' }],
                choices: [{ text: '"I\'m listening."', nextNode: 'teto_main' }]
            },
            'teto_main': {
                diaryEntry: "Teto wants me to be powerful... Power means money. Money means I don't have to worry about King Fimbul buying up all the damn tobacco leaves in the world. Maybe she has a point.",
                dialogue: [
                    { speaker: 'KasaneTeto', text: 'Your admin friend is about to have a... catastrophe. A server meltdown. I want you to be there to "welcome" the fleeing users. Poach them.' },
                    { speaker: 'KasaneTeto', text: 'He will be ruined. But you will be powerful.' },
                ],
                choices: [
                    { text: '"I\'ll do it. Power is what I want."', nextNode: 'pre_climax_betray' },
                    { text: '"I can\'t. He\'s my friend."', nextNode: 'pre_climax_defy' },
                ]
            },
            'pre_climax_betray': {
                dialogue: [{ speaker: 'KasaneTeto', text: 'Excellent. Stand by.' }],
                nextNode: 'pre_climax_end'
            },
            'pre_climax_defy': {
                dialogue: [{ speaker: 'KasaneTeto', text: 'Friendship? Pathetic. So be it.' }],
                nextNode: 'pre_climax_end'
            },
            'pre_climax_end': {
                dialogue: [],
                onComplete: () => { queueMessage('theorytoe', 1000, 'theorytoe_betrayal_start'); }
            },
            'theorytoe_betrayal_start': {
                dialogue: [
                    { speaker: 'TheoryToe', text: 'Milkman! My predictive models were correct! A hostile network event is imminent! My forensic analysis of the federation logs indicates a high probability of a backdoor exploit originating from Teto\'s node!' },
                    { speaker: 'TheoryToe', text: 'Fortunately, my superior intellect has already reverse-engineered a counter-exploit. A truly elegant piece of code. It will cost you a mere 10,000 dinars, a pittance for the preservation of your digital existence.' }
                ],
                choices: [
                    { text: 'Pay the 10,000 dinars. "Here you go."', nextNode: 'theorytoe_pay' },
                    { text: 'Refuse to pay. "I don\'t trust you."', nextNode: 'theorytoe_no_pay' }
                ]
            },
            'theorytoe_pay': {
                onChoice: (state) => { state.milkman.dinars -= 10000; state.plot.theoryToeBetrayed = true; },
                dialogue: [{ speaker: 'TheoryToe', text: 'A wise investment! Here is the code. Run it when the time is right. It will surely stop Teto!' }],
                nextNode: 'theorytoe_betrayal_end'
            },
            'theorytoe_no_pay': {
                dialogue: [{ speaker: 'TheoryToe', text: 'Your loss! You will regret spurning my intellect!' }],
                nextNode: 'theorytoe_betrayal_end'
            },
            'theorytoe_betrayal_end': {
                dialogue: [],
                onComplete: () => { queueMessage('ahab', 1000, 'climax_start_ahab'); }
            },
            'climax_start_ahab': {
                dialogue: [
                    { speaker: 'CaptainAhab', text: 'Lad, the time is now! Teto makes her move!' },
                    { speaker: 'CaptainAhab', text: 'But there is something you must know. The reason I hunt her. The reason I know her methods.' },
                    { speaker: 'CaptainAhab', text: 'Kasane Teto... is my daughter.' }
                ],
                nextNode: 'climax_ahab_reveal'
            },
            'climax_ahab_reveal': {
                dialogue: [
                    { speaker: 'CaptainAhab', text: 'She twisted my dream of a free network into this... this monster. The backdoor is her creation. TempusFugit is not a place, lad, it\'s a kill-switch. A protocol I designed to destroy her backdoor if she ever went too far.' },
                ],
                onComplete: () => { queueMessage('owl', 500, 'owl_twist'); }
            },
            'owl_twist': {
                dialogue: [
                    { speaker: 'Owl', text: 'Before you do anything stupid, Milkman. You should know why I have an interest in you.' },
                    { speaker: 'Owl', text: 'The user you replaced on this instance... the one who was "deactivated" right before you joined? That was my brother.' },
                    { speaker: 'Owl', text: 'Teto erased him because he wouldn\'t play her game. You just waltzed in and took his place. Now you get to decide the fate of the world he died for.' },
                ],
                nextNode: 'final_choice'
            },
            'final_choice': {
                diaryEntry: "So it all comes down to this. A power-hungry CEO, her crazy dad, a space wizard, and some anarchist. And I'm almost out of smokes. Fimbul's probably lighting up that city-cigar right now. Whatever I do, I'm doing it for one last decent drag.",
                onComplete: () => { queueMessage('zerocool', 1000, 'zerocool_final_offer'); }
            },
            'zerocool_final_offer': {
                timelinePost: { character: 'zerocool', text: 'The only winning move is not to play. Or, to flip the board over and set the pieces on fire. #HackThePlanet' },
                dialogue: [
                    { speaker: 'ZeroCool', text: 'Yo. Milkman. The big players are making their move. Boring.' },
                    { speaker: 'ZeroCool', text: 'Teto wants control. Ahab wants revenge. The corps want your data. I want chaos. Pure, beautiful chaos.' },
                    { speaker: 'ZeroCool', text: 'I have a virus that will wipe the entire Fediverse. Not reset it. Erase it. Every post, every user, every instance. A clean slate. The ultimate protest.' },
                    { speaker: 'ZeroCool', text: 'Help me launch it. Let\'s burn it all down.' },
                ],
                nextNode: 'final_choice_4'
            },
            'final_choice_4': {
                dialogue: [
                    { speaker: 'KasaneTeto', text: 'The migration is starting. Help me build the future, Milkman. A stable, orderly, profitable future. You will be a king.' },
                    { speaker: 'CaptainAhab', text: 'Help ME, lad! Help me burn it all down so something new and pure can grow from the ashes! For freedom!' },
                    { speaker: 'JudgeDread', text: 'There is another way. The path of the Jedi. Expose them both. Trust the people of the network to make their own choice.' },
                    { speaker: 'ZeroCool', text: 'Or, we can just delete the whole damn thing. Your call.' }
                ],
                choices: [
                    { text: 'Side with Teto. (The Golden Cage)', nextNode: 'ending_power' },
                    { text: 'Side with Ahab. (The Liberator)', nextNode: 'ending_integrity' },
                    { text: 'Side with Judge Dread. (The Ghost)', nextNode: 'ending_ghost' },
                    { text: 'Side with ZeroCool. (The Apocalypse)', nextNode: 'ending_chaos' }
                ]
            },
            'ending_power': {
                onChoice: (state) => { state.milkman.followers += 10000; state.plot.arc1_ending = 'power'; },
                dialogue: [{ speaker: 'System', text: 'You helped Teto. The migration was a success. The Fediverse was reborn, centralized and controlled...' }],
                nextNode: 'arc2_start'
            },
            'ending_integrity': {
                onChoice: (state) => { state.milkman.followers = 1; state.milkman.dinars = 0; state.plot.arc1_ending = 'integrity'; },
                dialogue: [{ speaker: 'System', text: 'You sided with Ahab. The network crashed... When it came back online, it was fractured, chaotic, but free...' }],
                nextNode: 'arc2_start'
            },
            'ending_ghost': {
                onChoice: (state) => { state.milkman.followers = 0; state.plot.arc1_ending = 'ghost'; },
                dialogue: [{ speaker: 'System', text: 'You sent all the logs to a journalist. The Fediverse erupted in war, fracturing into a thousand pieces...' }],
                nextNode: 'arc2_start'
            },
            'ending_chaos': {
                onChoice: (state) => { state.milkman.followers = 0; state.milkman.dinars = 0; state.plot.arc1_ending = 'chaos'; },
                dialogue: [{ speaker: 'System', text: 'You unleashed the virus. The effect was immediate and total. The Fediverse is gone... nothing left but silence.' }],
                nextNode: 'arc2_start'
            },
            // --- ARC 2 ---
            'arc2_start': {
                dialogue: [{ speaker: 'System', text: '--- ONE YEAR LATER ---' }],
                onComplete: () => {
                    // Reset some characters for the new arc
                    Object.keys(gameState.buddies).forEach(k => {
                        const buddy = gameState.buddies[k];
                        buddy.online = false;
                        buddy.hasNewMessage = false;
                        buddy.isVisible = false;
                        buddy.chatHistory = [];
                        buddy.completedNodes = new Set();
                    });
                    queueMessage('nierenstein', 1000);
                }
            },
            'nierenstein_intro': {
                diaryEntry: "A year later, and Fimbul's cigar is almost done. A pack of smokes costs 200 dinars. I'm smoking tea leaves wrapped in newspaper. This is fine.",
                timelinePost: { character: 'nierenstein', text: 'SPECTROBESFEST 2026 IS COMING! The ultimate celebration of the greatest game ever made! I am seeking influencers to promote this blessed event! #Spectrobes #NeverForget' },
                dialogue: [{ speaker: 'Nierenstein', text: 'You are Milkman. My research indicates you have... or had... influence. I require your services.' }, { speaker: 'Nierenstein', text: 'You will promote Spectrobesfest. In return, I will grant you 5,000 dinars. Enough for a few cartons of those cigarettes you seem to crave.' }],
                choices: [{ text: 'You got a deal.', nextNode: 'nierenstein_deal' }, { text: 'Get lost, shrimp.', nextNode: 'nierenstein_no_deal' }]
            },
            'nierenstein_deal': {
                onChoice: state => { state.milkman.dinars += 5000; },
                dialogue: [{ speaker: 'Nierenstein', text: 'Excellent. Do not disappoint me.' }],
                nextNode: 'nierenstein_end'
            },
            'nierenstein_no_deal': {
                dialogue: [{ speaker: 'Nierenstein', text: 'You will regret this slight.' }],
                nextNode: 'nierenstein_end'
            },
            'nierenstein_end': {
                dialogue: [],
                onComplete: () => {
                    addPostToTimeline('spectrefan', 'Nierenstein is a FAKE FAN. He doesn\'t even know the optimal fossil excavation route in Spectrobes: Beyond the Portals. Spectrobesfest will be a disaster. #RealFan');
                    setTimeout(() => {
                        addPostToTimeline('admin', 'Spectrobesfest has been officially announced! We are pleased to report Nierenstein was found deactivated in his wheelchair this morning. An investigation is underway.');
                        gameState.plot.nierenstein_murdered = true;
                        queueMessage('gumshoe', 2000);
                    }, 5000);
                }
            },
            'gumshoe_intro': {
                dialogue: [
                    { speaker: 'DetectiveGumshoe', text: 'The name\'s Gumshoe. I\'m looking into the Nierenstein case. Word on the wire is you were the last person he talked to.' },
                    { speaker: 'DetectiveGumshoe', text: 'This whole thing stinks worse than last week\'s chili dog. I\'m a private eye, but my jurisdiction is limited. I need a man on the inside. You help me, I\'ll make it worth your while.' }
                ],
                choices: [{ text: 'I\'ll help you.', nextNode: 'gumshoe_accept' }]
            },
            'gumshoe_accept': {
                dialogue: [{ speaker: 'DetectiveGumshoe', text: 'Good. The prime suspect is that fanatic kid. Go talk to him. See what he knows.' }],
                onComplete: () => {
                    queueMessage('spectrefan');
                    queueMessage('owl');
                    queueMessage('chadcorp');
                }
            },
            'spectrefan_intro': {
                dialogue: [{ speaker: 'SpectrobeFanatic117', text: 'What do you want? Come to gloat? I TOLD everyone Nierenstein was a fake! He probably tripped over his own wheelchair!' }],
                choices: [{ text: 'Where were you when it happened?', nextNode: 'spectrefan_alibi' }]
            },
            'spectrefan_alibi': {
                dialogue: [{ speaker: 'SpectrobeFanatic117', text: 'I was in the Spectrobes trivia competition! I won, obviously. Unlike that poser Nierenstein.' }],
                nextNode: 'spectrefan_end'
            },
            'spectrefan_end': { dialogue: [] },
            // ... more investigation nodes ...
        };

        function queueMessage(characterKey, delay, nodeKey) {
            setTimeout(() => {
                const buddy = gameState.buddies[characterKey];
                buddy.online = true;
                buddy.isVisible = true;
                buddy.hasNewMessage = true;
                if (nodeKey) {
                    buddy.currentNode = nodeKey;
                }
                updateBuddyList();
            }, delay);
        }

        function startConversation(characterKey) {
            const buddy = gameState.buddies[characterKey];
            if (!buddy.online) return;

            buddy.hasNewMessage = false;
            gameState.currentConversationKey = characterKey;

            chatWindowTitle.textContent = `IM with ${buddy.name}`;
            choicesContainer.innerHTML = '';
            statusText.textContent = `Connecting to ${buddy.name}...`;
            updateBuddyList();
            updateProfileWindow(characterKey);

            // Render chat history
            chatWindow.innerHTML = '';
            buddy.chatHistory.forEach(line => {
                const lineEl = document.createElement('p');
                lineEl.innerHTML = `<strong>${line.speaker}:</strong> ${line.text}`;
                chatWindow.appendChild(lineEl);
            });
            chatWindow.scrollTop = chatWindow.scrollHeight;

            // If the current node for this chat hasn't been completed, render it.
            if (!buddy.completedNodes.has(buddy.currentNode)) {
                renderNode(buddy.currentNode);
            } else {
                statusText.textContent = 'Conversation ended.';
            }
        }

        function addPostToTimeline(characterKey, postText, customUser = null) {
            let name = 'Anonymous';
            if (customUser) {
                name = customUser;
            } else if (characterKey === 'milkman') {
                name = 'Milkman';
            } else if (gameState.buddies[characterKey]) {
                name = gameState.buddies[characterKey].name;
            }

            if (timelineWindow.querySelector('.text-gray-500')) timelineWindow.innerHTML = '';
            const postEl = document.createElement('div');
            postEl.className = 'post';
            postEl.innerHTML = `<div class="post-header">${name}</div><div class="post-content">${postText}</div>`;
            timelineWindow.prepend(postEl);
        }

        function addRandomPost() {
            if (Math.random() > 0.65) { // 35% chance
                const post = randomPosts[Math.floor(Math.random() * randomPosts.length)];
                addPostToTimeline('random', post.text, post.user);
            }
        }

        function updateDiary(entryText) {
            if (diaryWindow.querySelector('.text-gray-500')) diaryWindow.innerHTML = '';
            const entryEl = document.createElement('div');
            entryEl.className = 'diary-entry';
            entryEl.innerHTML = `<p>${entryText}</p>`;
            diaryWindow.prepend(entryEl);
        }

        function updateUserProfile() {
            followerCountEl.textContent = `Followers: ${gameState.milkman.followers}`;
            dinarCountEl.textContent = `Dinars: ${gameState.milkman.dinars}`;
        }

        function updateProfileWindow(characterKey) {
            if (characterKey && gameState.buddies[characterKey]) {
                const buddy = gameState.buddies[characterKey];
                profileWindow.innerHTML = `
                        <div class="font-bold text-lg">${buddy.name}</div>
                        <div class="text-sm">Followers: ${buddy.followers}</div>
                        <p class="text-sm mt-2">${buddy.bio}</p>
                    `;
            } else {
                profileWindow.innerHTML = `<p class="text-gray-500">Select a buddy to view their profile.</p>`;
            }
        }

        function updateBuddyList() {
            buddyListContainer.innerHTML = '';
            const visibleBuddies = Object.entries(gameState.buddies).filter(([key, buddy]) => buddy.isVisible);

            visibleBuddies.sort(([keyA, a], [keyB, b]) => {
                if (a.hasNewMessage && !b.hasNewMessage) return -1;
                if (!a.hasNewMessage && b.hasNewMessage) return 1;
                if (a.online && !b.online) return -1;
                if (!a.online && b.online) return 1;
                return 0;
            });

            for (const [buddyKey, buddy] of visibleBuddies) {
                const buddyEl = document.createElement('div');
                buddyEl.textContent = buddy.name;
                buddyEl.dataset.buddy = buddyKey;

                let classes = 'buddy-list-item p-1';
                if (buddy.online) {
                    classes += ' online';
                    buddyEl.onclick = () => startConversation(buddyKey);
                } else {
                    classes += ' offline';
                }
                if (buddy.hasNewMessage) {
                    classes += ' flash';
                }
                buddyEl.className = classes;
                buddyListContainer.appendChild(buddyEl);
            }
        }

        async function renderNode(nodeKey) {
            const node = story[nodeKey];
            if (!node) return;

            const buddy = gameState.buddies[gameState.currentConversationKey];
            choicesContainer.innerHTML = '';

            if (node.onChoice) {
                node.onChoice(gameState);
                updateUserProfile();
            }

            if (node.timelinePost) {
                addPostToTimeline(node.timelinePost.character, node.timelinePost.text);
            }

            if (node.diaryEntry) {
                updateDiary(node.diaryEntry);
            }

            for (const line of node.dialogue) {
                await typeLine(line.speaker, line.text);
                if (line.speaker !== 'You' && line.speaker !== 'System') {
                    buddy.chatHistory.push({ speaker: line.speaker, text: line.text });
                }
            }

            if (node.choices) {
                statusText.textContent = 'Awaiting your response...';
                node.choices.forEach(choice => {
                    const button = document.createElement('button');
                    button.textContent = choice.text;
                    button.className = 'choice-button w-full text-left p-2 mt-1 text-black';
                    const currentChoice = choice;
                    button.onclick = async () => {
                        choicesContainer.innerHTML = ''; // Instantly clear choices
                        buddy.chatHistory.push({ speaker: 'You', text: currentChoice.text });
                        await typeLine('You', currentChoice.text);

                        buddy.completedNodes.add(nodeKey);
                        buddy.currentNode = currentChoice.nextNode;

                        renderNode(currentChoice.nextNode);
                    };
                    choicesContainer.appendChild(button);
                });
            } else {
                buddy.completedNodes.add(nodeKey);
                if (node.nextNode) {
                    buddy.currentNode = node.nextNode;
                    renderNode(node.nextNode);
                } else {
                    statusText.textContent = 'Conversation ended.';
                    if (node.onComplete) {
                        node.onComplete();
                    }
                    addRandomPost();
                }
            }
        }

        function typeLine(speaker, text) {
            return new Promise(resolve => {
                const typingDelay = 35;
                let charIndex = 0;

                const lineEl = document.createElement('p');
                lineEl.innerHTML = `<strong>${speaker}:</strong> `;
                chatWindow.appendChild(lineEl);
                chatWindow.scrollTop = chatWindow.scrollHeight;


                if (speaker !== 'You' && speaker !== 'System' && speaker !== 'Milkman') {
                    statusText.textContent = `${speaker} is typing...`;
                }

                function typeChar() {
                    if (charIndex < text.length) {
                        lineEl.innerHTML += text.charAt(charIndex);
                        charIndex++;
                        chatWindow.scrollTop = chatWindow.scrollHeight;
                        setTimeout(typeChar, typingDelay);
                    } else {
                        statusText.textContent = 'Message received.';
                        resolve();
                    }
                }
                typeChar();
            });
        }

        function jumpToArc2(ending) {
            // Reset core stats
            gameState.milkman = { influence: 0, hasTreasure: false, integrity: 10, followers: 0, dinars: 0 };
            diaryWindow.innerHTML = '';

            gameState.plot.arc1_ending = ending;
            const endingNode = story[`ending_${ending}`];
            if (endingNode.onChoice) {
                endingNode.onChoice(gameState);
            }
            updateUserProfile();

            // Clear all chat windows and history
            Object.keys(gameState.buddies).forEach(k => {
                const buddy = gameState.buddies[k];
                buddy.online = false;
                buddy.hasNewMessage = false;
                buddy.isVisible = false;
                buddy.chatHistory = [];
                buddy.completedNodes = new Set();
            });

            chatWindow.innerHTML = '';
            choicesContainer.innerHTML = '';
            updateProfileWindow(null);

            // Set up diary based on ending
            const diaryEntries = {
                power: "A year in Teto's world. I'm 'powerful', sure, but it's hollow. Every post, every thought, filtered. At least the smokes are free.",
                integrity: "The 'Great Reset' they called it. Wiped everything. I'm a nobody again, broke as a joke. But the network feels... quiet. Clean. Maybe Ahab was right.",
                ghost: "Been a year since I dropped the bomb and cashed out. The old Fediverse is a warzone. I watch from the sidelines, comfortable. And completely alone.",
                chaos: "It's been a year of silence. We actually did it. Wiped the whole damn thing. Sometimes I wonder if it was the right call. Then I remember what it was like, and I know it was."
            };
            updateDiary(diaryEntries[ending]);

            renderNode('arc2_start');
            debugModal.classList.add('hidden');
        }

        function initializeGame() {
            updateBuddyList();
            updateUserProfile();
            document.addEventListener('keydown', (e) => {
                if (e.key.toLowerCase() === 'p') {
                    debugModal.classList.remove('hidden');
                }
            });
        }

        initializeGame();
    </script>
</body>
</html>
