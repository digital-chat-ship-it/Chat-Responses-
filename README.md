# Chat-Responses-

step 1


/* 2-Box Parallel Side-by-Side Mode (for Sheet 3) */


/* 2-Box Parallel Side-by-Side Mode (for Sheet 3) */
        .grid-container.two-box-sheet #col2,
        .grid-container.two-box-sheet #card2,
        .grid-container.two-box-sheet #row1,
        .grid-container.two-box-sheet #card3,
        .grid-container.two-box-sheet #card4,
        .grid-container.two-box-sheet #card5,
        .grid-container.two-box-sheet .spacer {
            display: none !important;
        }

        .grid-container.two-box-sheet #col2,
        .grid-container.two-box-sheet #card2,
        .grid-container.two-box-sheet #row1,
        .grid-container.two-box-sheet #card3,
        .grid-container.two-box-sheet #card4,
        .grid-container.two-box-sheet #card5,
        .grid-container.two-box-sheet .spacer {
            display: none !important;
        }


step 2 

function createBlankTwentyOptions() {
            return Array.from({ length: 20 }, (_, i) => ({
                name: `Option ${i + 1}`,
                text: ''
            }));
        }

        function createThirdSheet() {
            const sheet1 = state.sheets[0];
            return {
                id: 3,
                name: 'Sheet 3',
                defaultName: 'Sheet 3',
                color: '#8b5cf6',
                boxCount: 2,
                syncVarsFromSheetId: 1,
                boxHeadings: ['Box 1', 'Box 2', '', '', '', ''],
                boxSequences: {
                    "0": createBlankTwentyOptions(),
                    "1": createBlankTwentyOptions(),
                    "2": [],
                    "3": [],
                    "4": [],
                    "5": []
                },
                cardColors: ['#008a00', '#0284c7', '', '', '', ''],
                contents: ['', '', '', '', '', ''],
                selections: [0, 0, 0, 0, 0, 0],
                edits: {},
                variables: sheet1 && sheet1.variables ? { ...sheet1.variables } : {}
            };
        }

        /* ===== PRE-CONFIGURED 3 SHEETS ON STARTUP ===== */
        function setupDefaultSheets() {
            const s1 = createNewSheet(1, 'Claim Intake', [0, 0, 0, 0, 0, 0]);

            const transferNotes = [
                "Hello <b>{{customer_name}}</b>. Thank you for contacting TD Insurance Claims Response Operations. My name is <b>{{agent_name}}</b>. I see you've reached our claims desk. How can I assist you with your policy or vehicle today?",
                "Could you please confirm if you are looking to make changes to an active policy, submit a premium payment, update banking details, or obtain proof of insurance (pink card)?",
                "I see our automated system directed your chat to <b>Claims Operations</b> by mistake. Because your inquiry relates to <b>policy endorsement, billing, or general changes</b>, I will connect you directly with our <b>Client Advice Centre (CAC)</b> right now.",
                "If you need policy documents or pink slips urgently, you can also download them instantly via the <b>TD Insurance App</b>. If you prefer to speak by phone, our Client Advice Centre is available at <b>1-866-454-8910</b> (Mon–Fri 8 AM–8 PM, Sat 9 AM–4 PM).",
                "Please stay connected. I am transferring your chat along with your account notes directly to the <b>Client Advice Centre</b> so you will not have to repeat your details to the next advisor.",
                "Your chat is being routed now. An advisor will be with you shortly. Thank you for choosing TD Insurance, and have a wonderful day ahead!"
            ];

            const s2 = createNewSheet(2, 'Transfers & CAC Handoff', [17, 19, 0, 10, 11, 9], transferNotes);
            s2.color = '#0284c7';

            state.sheets = [s1, s2];
            const s3 = createThirdSheet();
            state.sheets.push(s3);

            state.activeSheetId = 1;
            state.sheetCounter = 3;
        }

        function restoreBrowserData() {
            try {
                if (seed.DEFAULT_VARS) DEFAULT_VARS = seed.DEFAULT_VARS;

                const saved = localStorage.getItem(STORAGE_KEY);
                if (!saved) {
                    if (seed.savedState && Array.isArray(seed.savedState.sheets) && seed.savedState.sheets.length) {
                        state = { ...state, ...seed.savedState };
                    } else {
                        setupDefaultSheets();
                    }
                } else {
                    const data = JSON.parse(saved);
                    if (data.boxHeadings) boxHeadings = data.boxHeadings;
                    if (data.cardColors) cardColors = data.cardColors;
                    if (data.boxSequences) boxSequences = data.boxSequences;
                    if (data.DEFAULT_VARS) DEFAULT_VARS = data.DEFAULT_VARS;
                    if (data.state) state = { ...state, ...data.state };
                }

                if (!state.sheets || !state.sheets.length) {
                    setupDefaultSheets();
                }

                // Ensure Sheet 3 exists even if you already had old data in browser storage
                if (!state.sheets.some(s => s.boxCount === 2)) {
                    state.sheets.push(createThirdSheet());
                    state.sheetCounter = Math.max(state.sheetCounter || 3, state.sheets.length);
                }

                state.sheets.forEach(s => ensureSheetStructure(s));

                if (!state.dark) document.body.classList.add('light-mode');
                applyLayout(state.layout || '3x2');
                if (state.compactHeaders) {
                    document.body.classList.add('compact-headers');
                    document.getElementById('compactHeaderBtn')?.classList.add('active-state');
                }
                setGlobalFontSize(state.fontSize || 13.5);
                applyAllCardColors();
            } catch (err) {
                console.warn('Error restoring data:', err);
                setupDefaultSheets();
            }
        }


step 3 


function loadSheet(id) {
            const sheet = state.sheets.find(s => s.id === id) || state.sheets[0];
            ensureSheetStructure(sheet);

            // Pull variables from the 1st sheet (Sheet 1) into Sheet 3
            if (sheet.syncVarsFromSheetId) {
                const firstSheet = state.sheets.find(s => s.id === sheet.syncVarsFromSheetId) || state.sheets[0];
                if (firstSheet) {
                    sheet.variables = { ...(firstSheet.variables || {}) };
                }
            }

            // Toggle 2-box side-by-side class and refresh grid layout
            grid.classList.toggle('two-box-sheet', sheet.boxCount === 2);
            applyLayout(state.layout || '3x2');

            // Keep activeBoxIdx inside 0..1 on a 2-box sheet
            if (sheet.boxCount === 2 && activeBoxIdx > 1) {
                setActiveBoxIndex(0);
            }

            initHeadings();
            applyAllCardColors();

            editors.forEach((ed, i) => {
                ed.innerHTML = sheet.contents[i] || '';
                upFill(ed, sheet.variables);
                boxHistories[i] = [ed.innerHTML];
                let sel = sheet.selections[i] || 0;
                sequenceIndices[i] = sel;
                checkForVariables(i);
            });
            initDropdowns();
            updateAllStats();
            renderDrawerVariables();
            refreshLiveToolbar();
            if (!searchBarPanel.classList.contains('hidden')) runSearch();
        }

------

// DYNAMIC SHEET NAME (keep Sheet 3's own tab title)
            if (key === 'customer_name' && !sheet.syncVarsFromSheetId) {
                sheet.name = value.trim() || (sheet.defaultName || 'Claim Intake');
                renderSheetTabs();
            }


step 4 

function applyLayout(mode) {
            state.layout = mode === '1col' ? '1col' : '3x2';
            grid.classList.remove('layout-3x2', 'layout-1col');
            grid.classList.add(`layout-${state.layout}`);
            const isTwoBox = getActiveSheet()?.boxCount === 2;

            if (state.layout === '3x2') {
                if (isTwoBox) {
                    grid.style.gridTemplateColumns = '1fr 6px 1fr';
                    grid.style.gridTemplateRows = '1fr';
                } else {
                    grid.style.gridTemplateColumns = '1fr 6px 1fr 6px 1fr';
                    grid.style.gridTemplateRows = '1fr 6px 1fr';
                }
            } else {
                grid.style.gridTemplateColumns = '';
                grid.style.gridTemplateRows = '';
            }
        }


------

1.



if (activeDrag === 'col1') {
                const isTwoBox = getActiveSheet()?.boxCount === 2;
                const left = Math.max(130, Math.min(rect.width - (isTwoBox ? 140 : 280), e.clientX - rect.left));
                grid.style.gridTemplateColumns = isTwoBox
                    ? `${left}px 6px 1fr`
                    : `${left}px 6px 1fr 6px 1fr`;
            }



step 5



const exportPayload = {
                boxHeadings,
                defaultCardColors: cardColors,
                boxSequences,
                DEFAULT_VARS,
                savedState: state
            };
            clone.querySelector('#seedData').textContent = JSON.stringify(exportPayload, null, 4).replace(/<\//g, '<\\/');


------

step 6 


// Cycle options in Active Box (Alt+Right / Alt+Left or Alt+N / Alt+P)
            if (e.altKey && (e.key === 'ArrowRight' || e.key === 'ArrowLeft' || e.code === 'ArrowRight' || e.code === 'ArrowLeft' || e.key === 'n' || e.key === 'p')) {
                e.preventDefault();
                e.stopPropagation();
                const maxBox = (getActiveSheet()?.boxCount || 6) - 1;
                const targetBox = Math.min(activeBoxIdx, maxBox);
                if (e.key === 'ArrowRight' || e.code === 'ArrowRight' || e.key === 'n') {
                    nextTextInBox(targetBox);
                } else {
                    prevTextInBox(targetBox);
                }
                toast(`Box ${targetBox + 1}: Option ${sequenceIndices[targetBox] + 1}`);
            }


