<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Confirmação de Presença - Casamento</title>
  <style>
    body { font-family: Arial, sans-serif; margin:0; padding:0; background:#f8f9fa; }
    header { background:#c2185b; color:#fff; text-align:center; padding:20px; }
    header h1 { margin:0; font-size:28px; }
    header h2 { margin:5px 0 0; font-weight:normal; font-size:20px; }
    main { padding:20px; max-width:900px; margin:auto; }
    .card { background:#fff; padding:20px; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,.2); margin-bottom:20px; }
    label { display:block; margin-top:10px; font-weight:bold; }
    input, select, textarea { width:100%; padding:10px; margin-top:5px; border:1px solid #ccc; border-radius:5px; }
    button { margin-top:15px; padding:10px 20px; background:#c2185b; color:#fff; border:none; border-radius:5px; cursor:pointer; }
    button:hover { background:#a3154a; }
    footer { text-align:center; padding:15px; background:#eee; color:#555; margin-top:30px; }
    .familia-item { border:1px solid #ddd; padding:10px; border-radius:8px; margin-bottom:15px; }
    .member-item { display:flex; justify-content:space-between; align-items:center; margin-top:8px; }
    .action-btn { margin-left:5px; padding:5px 10px; border:none; border-radius:5px; cursor:pointer; }
    .btn-danger { background:#e53935; color:#fff; }
    .btn-success { background:#43a047; color:#fff; }
    .btn-info { background:#1e88e5; color:#fff; }
    .tipo-convidado { padding:2px 6px; border-radius:4px; font-size:12px; color:#fff; }
    .adulto { background:#00796b; }
    .crianca { background:#0288d1; }
    .estatisticas p { margin:5px 0; }
  </style>
</head>
<body>

<header>
  <h1>Confirmação de Presença</h1>
  <h2>17/01/2026 em Garça</h2>
</header>

<main>
  <!-- ÁREA DO CONVIDADO -->
  <section id="convidado" class="card">
    <h2>Área do Convidado</h2>
    <label for="familia">Selecione sua família</label>
    <select id="familia">
      <option value="">-- Selecione --</option>
    </select>
    <label for="telefone">Telefone (WhatsApp)</label>
    <input type="text" id="telefone" placeholder="(00) 00000-0000">
    <label for="mensagem">Mensagem aos noivos</label>
    <textarea id="mensagem" rows="3"></textarea>
    <button id="btnConfirmarPresenca">Confirmar Presença</button>
    <p id="msgStatus"></p>
  </section>

  <!-- ÁREA ADMINISTRATIVA -->
  <section id="admin" class="card">
    <h2>Área Administrativa</h2>
    <input type="password" id="senha" placeholder="Digite a senha">
    <button id="btnLogin">Entrar</button>
    <div id="adminPanel" style="display:none; margin-top:20px;">
      <h3>Estatísticas</h3>
      <div class="estatisticas" id="estatisticas"></div>

      <h3>Gerenciar Famílias</h3>
      <button class="btn-info" onclick="adicionarFamilia()">Adicionar Família</button>
      <div style="margin: 15px 0;">
        <input type="text" id="busca-familia" placeholder="🔍 Buscar família..." 
               style="width:100%; padding:10px; border:1px solid #ccc; border-radius:5px;">
      </div>
      <div id="admin-familias"></div>

      <h3>Exportar Dados</h3>
      <button class="btn-success" onclick="exportarJSON()">Exportar JSON</button>
      <button class="btn-info" onclick="exportarCSV()">Exportar CSV</button>
    </div>
  </section>
</main>

<footer>
  <p>&copy; 2025 Casamento – Contato: (14) 98116-4925</p>
</footer>

<script>
  let familias = {
    "1": { nome:"Família Silva", telefone:"", membros:[{id:1,nome:"João",tipo:"adulto",confirmado:false},{id:2,nome:"Maria",tipo:"adulto",confirmado:false}] },
    "2": { nome:"Família Santos", telefone:"", membros:[{id:1,nome:"Carlos",tipo:"adulto",confirmado:false},{id:2,nome:"Ana",tipo:"crianca",confirmado:false}] }
  };

  const selectFamilia=document.getElementById("familia");
  const telefoneInput=document.getElementById("telefone");
  const mensagemInput=document.getElementById("mensagem");
  const btnConfirmarPresenca=document.getElementById("btnConfirmarPresenca");
  const msgStatus=document.getElementById("msgStatus");
  const senhaInput=document.getElementById("senha");
  const btnLogin=document.getElementById("btnLogin");
  const adminPanel=document.getElementById("adminPanel");
  const adminFamilias=document.getElementById("admin-familias");
  const estatisticasDiv=document.getElementById("estatisticas");
  const inputBusca=document.getElementById("busca-familia");

  /* ===== Persistência Local ===== */
  function salvarDados(){ localStorage.setItem("dadosConvidados", JSON.stringify(familias)); }
  function carregarDados(){ const dados=localStorage.getItem("dadosConvidados"); if(dados){ familias=JSON.parse(dados);} }

  /* ===== Convidado ===== */
  function carregarFamiliasSelect(){
    selectFamilia.innerHTML='<option value="">-- Selecione --</option>';
    for(const [id,familia] of Object.entries(familias)){
      const opt=document.createElement("option");
      opt.value=id; opt.textContent=familia.nome;
      selectFamilia.appendChild(opt);
    }
  }

  telefoneInput.addEventListener("input",function(){
    let v=this.value.replace(/\D/g,'');
    if(v.length>11) v=v.slice(0,11);
    if(v.length>6) this.value=`(${v.slice(0,2)}) ${v.slice(2,7)}-${v.slice(7)}`;
    else if(v.length>2) this.value=`(${v.slice(0,2)}) ${v.slice(2)}`;
    else this.value=v;
  });

  btnConfirmarPresenca.addEventListener("click",()=>{
    const familiaId=selectFamilia.value;
    const tel=telefoneInput.value.trim();
    const msg=mensagemInput.value.trim();
    if(!familiaId){ alert("Selecione sua família!"); return; }
    familias[familiaId].telefone=tel;
    familias[familiaId].membros.forEach(m=>m.confirmado=true);
    salvarDados();
    msgStatus.textContent="🎉 Presença confirmada! Obrigado!";
    msgStatus.style.color="green";
    console.log("Mensagem:",msg);
    carregarDadosAdministracao();
  });

  /* ===== Administração ===== */
  btnLogin.addEventListener("click",()=>{
    if(senhaInput.value==="noivos123"){ adminPanel.style.display="block"; carregarDadosAdministracao();}
    else alert("Senha incorreta!");
  });

  inputBusca.addEventListener("input",()=>{ carregarDadosAdministracao(inputBusca.value.toLowerCase()); });

  function carregarDadosAdministracao(filtro=""){
    adminFamilias.innerHTML="";
    for(const [id,familia] of Object.entries(familias)){
      if(filtro && !familia.nome.toLowerCase().includes(filtro) && !(familia.telefone||"").toLowerCase().includes(filtro)){ continue; }
      const div=document.createElement("div"); div.className="familia-item";
      let membrosHTML="";
      familia.membros.forEach(m=>{
        membrosHTML+=`
          <div class="member-item">
            <div>${m.nome} <span class="tipo-convidado ${m.tipo}">${m.tipo==="adulto"?"Adulto":"Criança"}</span> - ${m.confirmado?"Confirmado":"Pendente"}</div>
            <div>
              <select class="tipo-select" data-familia="${id}" data-membro="${m.id}">
                <option value="adulto" ${m.tipo==="adulto"?"selected":""}>Adulto</option>
                <option value="crianca" ${m.tipo==="crianca"?"selected":""}>Criança</option>
              </select>
              <button class="action-btn btn-danger" onclick="removerMembro('${id}',${m.id})">Remover</button>
            </div>
          </div>`;
      });
      div.innerHTML=`
        <h3>${familia.nome} (ID:${id})</h3>
        <p><b>Telefone:</b> ${familia.telefone||"Não informado"}</p>
        ${membrosHTML}
        <div><button class="action-btn btn-success" onclick="adicionarMembro('${id}')">Adicionar Membro</button>
        <button class="action-btn btn-danger" onclick="removerFamilia('${id}')">Remover Família</button></div>`;
      adminFamilias.appendChild(div);
    }
    document.querySelectorAll(".tipo-select").forEach(sel=>{
      sel.addEventListener("change",function(){
        const f=this.dataset.familia; const m=parseInt(this.dataset.membro);
        familias[f].membros.find(x=>x.id===m).tipo=this.value; salvarDados();
      });
    });
    atualizarEstatisticas();
  }

  function atualizarEstatisticas(){
    let total=0,confirmados=0,adultos=0,criancas=0;
    for(const familia of Object.values(familias)){
      familia.membros.forEach(m=>{
        total++; if(m.confirmado)confirmados++;
        if(m.tipo==="adulto") adultos++; else criancas++;
      });
    }
    estatisticasDiv.innerHTML=`
      <p><b>Total convidados:</b> ${total}</p>
      <p><b>Confirmados:</b> ${confirmados}</p>
      <p><b>Adultos:</b> ${adultos}</p>
      <p><b>Crianças:</b> ${criancas}</p>`;
  }

  /* ===== Ações ===== */
  function adicionarFamilia(){
    const nome=prompt("Nome da família:"); if(!nome) return;
    const id=Date.now().toString(); familias[id]={nome,telefone:"",membros:[]}; salvarDados(); carregarFamiliasSelect(); carregarDadosAdministracao();
  }
  function adicionarMembro(fid){
    const nome=prompt("Nome do membro:"); if(!nome)return;
    const tipo=prompt("Tipo (adulto/crianca):","adulto"); 
    familias[fid].membros.push({id:Date.now(),nome,tipo,confirmado:false});
    salvarDados(); carregarDadosAdministracao();
  }
  function removerMembro(fid,mid){ familias[fid].membros=familias[fid].membros.filter(m=>m.id!==mid); salvarDados(); carregarDadosAdministracao(); }
  function removerFamilia(fid){ if(confirm("Remover família?")){ delete familias[fid]; salvarDados(); carregarFamiliasSelect(); carregarDadosAdministracao(); } }

  window.adicionarFamilia=adicionarFamilia; window.adicionarMembro=adicionarMembro; window.removerMembro=removerMembro; window.removerFamilia=removerFamilia;

  /* ===== Exportação ===== */
  function exportarJSON(){ const blob=new Blob([JSON.stringify(familias,null,2)],{type:"application/json"}); const a=document.createElement("a"); a.href=URL.createObjectURL(blob); a.download="convidados.json"; a.click();}
  function exportarCSV(){ 
    let linhas=["ID Família,Nome Família,Telefone,ID Membro,Nome Membro,Tipo,Confirmado"];
    for(const [fid,f] of Object.entries(familias)){ f.membros.forEach(m=>{linhas.push(`${fid},"${f.nome}","${f.telefone}",${m.id},"${m.nome}",${m.tipo},${m.confirmado}`);});}
    const blob=new Blob([linhas.join("\n")],{type:"text/csv"}); const a=document.createElement("a"); a.href=URL.createObjectURL(blob); a.download="convidados.csv"; a.click();
  }
  window.exportarJSON=exportarJSON; window.exportarCSV=exportarCSV;

  /* ===== Inicializar ===== */
  carregarDados(); carregarFamiliasSelect();
</script>

</body>
</html>
