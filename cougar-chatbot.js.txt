(function () {
  "use strict";

  const careers = {
    software: { title: "Software Developer", text: "A strong fit if you enjoy building, debugging, and turning ideas into working applications." },
    systems: { title: "Business / Systems Analyst", text: "A strong fit if you enjoy asking questions, improving processes, and connecting business needs with technology." },
    data: { title: "Data Analyst", text: "A strong fit if you enjoy finding patterns, checking details, and explaining what numbers mean." },
    cyber: { title: "Cybersecurity Analyst", text: "A strong fit if you enjoy investigating risks, protecting systems, and solving careful technical puzzles." },
    project: { title: "IT Project Manager", text: "A strong fit if you enjoy organizing teams, tracking progress, and helping people deliver work together." },
    product: { title: "UX / Product", text: "A strong fit if you enjoy understanding users, designing experiences, and deciding what should be built." },
    erp: { title: "ERP / Systems Consultant", text: "A strong fit if you enjoy client work, business processes, and configuring large enterprise platforms." },
    cloud: { title: "Cloud / Infrastructure", text: "A strong fit if you enjoy reliable systems, automation, networking, and troubleshooting." }
  };

  const quiz = [
    ["Which project sounds most enjoyable?", [["Building an app",{software:3,cloud:1}],["Finding insights in data",{data:3,systems:1}],["Improving a business process",{systems:3,erp:2}],["Testing system security",{cyber:3,cloud:1}]]],
    ["What do you naturally notice first?", [["Broken code",{software:3}],["Numbers that do not make sense",{data:3}],["A confusing user process",{product:3,systems:2}],["A risk others overlooked",{cyber:3,project:1}]]],
    ["How would you spend an internship day?", [["Coding and debugging",{software:3}],["Interviewing users or clients",{product:3,erp:2}],["Building a dashboard",{data:3}],["Coordinating a team",{project:3,systems:1}]]],
    ["Which tools interest you most?", [["React, Python or Java",{software:3}],["SQL, Tableau or Power BI",{data:3}],["AWS, Docker or Linux",{cloud:3,cyber:1}],["Figma, Jira or Salesforce",{product:2,project:2,erp:2}]]],
    ["What role do you take on a team?", [["The builder",{software:2,cloud:2}],["The organizer",{project:3}],["The translator",{systems:3,erp:2}],["The detail checker",{data:2,cyber:2}]]],
    ["Which result would make you proudest?", [["People use what I built",{software:3,product:2}],["My analysis improves a decision",{data:3,systems:1}],["A client rollout succeeds",{erp:3,project:2}],["A system stays safe and reliable",{cyber:2,cloud:3}]]]
  ];

  const interviews = {
    software: [["How would you debug an API endpoint returning a 500 error?","Reproduce the request, inspect logs, isolate the failure, test the fix, and prevent regression."],["Tell me about a technical project and one tradeoff you made.","Use Situation–Task–Action–Result, then explain why your tradeoff fit the constraints."]],
    systems: [["A stakeholder requests a dashboard but cannot explain why. What do you do?","Ask who uses it, what decision it supports, which metrics matter, and how success will be tested."],["Write a user story for resetting a forgotten password.","Use: As a [user], I want [goal], so that [value], followed by testable acceptance criteria."]],
    data: [["Which SQL join returns every customer and any orders they may have?","A LEFT JOIN from customers to orders preserves every customer and returns NULL when no order matches."],["Your total does not match Finance's report. How do you investigate?","Compare definitions, filters, dates, time zones, data freshness, row grain, and duplicate-producing joins."]],
    cyber: [["An employee clicked a suspicious link. What are your first actions?","Follow the incident plan: record details, contain risk, preserve evidence, assess credentials, escalate, and document."],["Explain hashing versus encryption.","Encryption is reversible with a key; hashing is designed to be one-way and is used for integrity and password storage."]]
  };

  const assetBase = document.currentScript?.src ? new URL(".", document.currentScript.src).href : "";
  const root = document.createElement("div");
  root.id = "is-cougar-chatbot";
  root.innerHTML = `
    <button class="icc-launch" aria-label="Open Cougar Career Coach">
      <img src="${assetBase}cougar-guide.png" alt="8-bit cougar mascot"><span>ASK THE COUGAR</span>
    </button>
    <section class="icc-panel" hidden aria-label="Cougar Career Coach">
      <header><img src="${assetBase}cougar-guide.png" alt=""><div><b>Cougar Career Coach</b><small>IS careers & interview prep</small></div><button class="icc-close" aria-label="Close chatbot">×</button></header>
      <div class="icc-body"></div>
    </section>`;
  document.body.appendChild(root);

  const launch = root.querySelector(".icc-launch");
  const panel = root.querySelector(".icc-panel");
  const header = root.querySelector("header");
  const status = root.querySelector("header small");
  const body = root.querySelector(".icc-body");
  let quizIndex = 0, scores = {}, interviewRole = "software", interviewIndex = 0;

  function talk(next) {
    header.classList.add("icc-talking");
    status.textContent = "Typing a response…";
    body.innerHTML = '<div class="icc-typing" role="status" aria-label="Cougar is typing"><i></i><i></i><i></i></div>';
    window.setTimeout(() => {
      header.classList.remove("icc-talking");
      status.textContent = "IS careers & interview prep";
      next();
    }, 850);
  }

  function menu() {
    body.innerHTML = `<div class="icc-bubble">Hey! I can help you find an IS career that fits you or practice an interview.</div>
      <button data-action="quiz">✨ Take the career-fit quiz <span>›</span></button>
      <button data-action="interview">💬 Practice an interview <span>›</span></button>
      <p class="icc-note">Works without an API key.</p>`;
  }

  function quizQuestion() {
    const item = quiz[quizIndex];
    body.innerHTML = `<button class="icc-back" data-action="menu">← Menu</button>
      <div class="icc-progress"><span style="width:${((quizIndex+1)/quiz.length)*100}%"></span></div>
      <div class="icc-bubble"><b>Question ${quizIndex+1} of ${quiz.length}</b><br>${item[0]}</div>
      <div class="icc-options">${item[1].map((option,i)=>`<button data-quiz="${i}">${option[0]} <span>›</span></button>`).join("")}</div>`;
  }

  function quizResults() {
    const top = Object.entries(scores).sort((a,b)=>b[1]-a[1]).slice(0,3);
    body.innerHTML = `<button class="icc-back" data-action="menu">← Menu</button>
      <div class="icc-bubble"><b>Your strongest matches</b><br>Based on your interests and preferred work style:</div>
      <div class="icc-results">${top.map(([id],i)=>`<article><strong>#${i+1}</strong><div><b>${careers[id].title}</b><p>${careers[id].text}</p></div></article>`).join("")}</div>
      <button data-action="quiz">Retake quiz <span>↻</span></button>`;
  }

  function interview() {
    const q = interviews[interviewRole][interviewIndex];
    body.innerHTML = `<button class="icc-back" data-action="menu">← Menu</button>
      <label>Interview role<select class="icc-role">${Object.keys(interviews).map(id=>`<option value="${id}" ${id===interviewRole?"selected":""}>${careers[id].title}</option>`).join("")}</select></label>
      <div class="icc-bubble"><b>Question ${interviewIndex+1} of ${interviews[interviewRole].length}</b><br>${q[0]}</div>
      <textarea rows="5" placeholder="Type your answer here…"></textarea>
      <button data-action="review">Review my answer <span>›</span></button>`;
  }

  function review() {
    const q = interviews[interviewRole][interviewIndex];
    body.innerHTML += `<div class="icc-feedback"><b>Coach's checklist</b><p>${q[1]}</p><button data-action="next">Next question <span>›</span></button></div>`;
    body.querySelector('[data-action="review"]').remove();
  }

  root.addEventListener("click", (event) => {
    const target = event.target.closest("button");
    if (!target) return;
    if (target === launch) { launch.hidden = true; panel.hidden = false; menu(); return; }
    if (target.classList.contains("icc-close")) { panel.hidden = true; launch.hidden = false; return; }
    const action = target.dataset.action;
    if (action === "menu") talk(menu);
    if (action === "quiz") { quizIndex=0; scores={}; talk(quizQuestion); }
    if (action === "interview") talk(interview);
    if (action === "review") {
      if (!body.querySelector("textarea").value.trim()) return;
      talk(() => { interview(); review(); });
    }
    if (action === "next") { interviewIndex=(interviewIndex+1)%interviews[interviewRole].length; talk(interview); }
    if (target.dataset.quiz !== undefined) {
      const option = quiz[quizIndex][1][Number(target.dataset.quiz)];
      Object.entries(option[1]).forEach(([id,value]) => scores[id]=(scores[id]||0)+value);
      talk(() => { quizIndex++; quizIndex < quiz.length ? quizQuestion() : quizResults(); });
    }
  });

  root.addEventListener("change", (event) => {
    if (event.target.classList.contains("icc-role")) { interviewRole=event.target.value; interviewIndex=0; talk(interview); }
  });
})();
