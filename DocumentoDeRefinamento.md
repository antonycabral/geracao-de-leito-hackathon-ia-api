1. Visão Geral do Sistema (The BluePrint)O sistema operará sobre uma Máquina de Estados de Localização e Paciente. O objetivo é que o "Gêmeo Digital" do hospital saiba exatamente onde o paciente está e qual o próximo passo da sua jornada.2. Estrutura Hierárquica (Geolocalização Interna)Para que os relatórios e a logística funcionem, o sistema deve respeitar a árvore de dependências:Unidade: Hospital CentralBloco: Bloco A (Internação), Bloco B (Emergência/Triagem)Andar: 1º ao 5º AndarAla/Setor: Ala Sul (Cardiologia), Ala Norte (Infectologia)Leito: Identificador único (Ex: 302-A).3. Matriz de Eventos e Estados do PacienteAlém do status do leito, agora monitoramos o Status do Paciente dentro do leito:EventoAção no SistemaImpacto na LogísticaTriagemRegistro inicial (Manchester)Reserva técnica de leito por criticidade.MedicaçãoStatus: EM_MEDICACAOBloqueio de transporte para exames durante o gotejamento.Exame ExternoStatus: EM_EXAMELeito fica em modo OCUPADO_AUSENTE (sinaliza que o paciente volta).Visita MédicaStatus: AGUARDANDO_VISITAPriorização de informações no dashboard para o médico.Previsão de AltaData Estática + NotasGatilho para o "Pré-Giro" de leito.4. Requisitos Funcionais Detalhados (RF)RF.01 - Triagem e Admissão: O sistema deve capturar dados da triagem (queixa, sinais vitais e cor de risco) e sugerir o leito disponível mais próximo ou adequado ao perfil.RF.02 - Cronograma de Cuidados (Timeline): Exibir uma linha do tempo para cada paciente contendo:Horários de medicamentos (previsto vs. realizado).Status de exames (Solicitado -> Em Transporte -> Realizado).Previsão de visita médica (Baseada na escala do dia).RF.03 - Gestão de Prorrogação de Alta: Ao atingir a data prevista, se a alta não for confirmada, o médico deve obrigatoriamente inserir uma justificativa para a prorrogação.RF.04 - Portal do Acompanhante: Dashboard simplificado via Web que exibe o status atual do paciente (ex: "Em medicação", "Em exame", "Aguardando alta") sem expor dados sensíveis ou diagnósticos detalhados.RF.05 - Notificação de Limpeza de Emergência: Botão de pânico na enfermaria para acionar higienização imediata (vômito, fluidos) mantendo o paciente no leito.5. Regras de Negócio Críticas (RN)RN.01 - Conflito de Agenda: O sistema deve alertar se um exame for marcado para o mesmo horário de uma medicação endovenosa de ciclo fixo.RN.02 - Trava de Limpeza: Um leito não pode ser marcado como LIVRE se a tarefa de higienização não tiver o checklist de "Roupa de Cama" marcado.RN.03 - Visibilidade do Acompanhante: O acesso do acompanhante é revogado automaticamente 2 horas após a alta definitiva do paciente.RN.04 - Prioridade de Higienização: Chamados de limpeza em áreas críticas (UTI/Emergência) furam a fila de chamados de alas comuns.RN.05 - Registro de Notas: Toda alteração na data de previsão de alta deve gerar um log imutável com ID do médico e justificativa.6. Modelagem de Dados Orientada a FHIR (Contrato JSON)Para os novos itens, usaremos estes recursos:MedicationAdministration: Para o registro de quando o medicamento foi dado.ServiceRequest: Para o pedido de exame.Appointment: Para a agenda de visitas médicas ou exames.Communication: Para os comentários/notas da enfermaria e médicos.Exemplo de Objeto de Status (Timeline)JSON{
  "resourceType": "Bundle",
  "type": "collection",
  "entry": [
    {
      "resource": {
        "resourceType": "Task",
        "description": "Visita Médica de Rotina",
        "status": "requested", // ou "in-progress"
        "owner": { "display": "Dr. Silva - Cardiologista" }
      }
    },
    {
      "resource": {
        "resourceType": "MedicationAdministration",
        "status": "completed",
        "effectiveDateTime": "2026-03-11T14:00:00Z",
        "medicationCodeableConcept": { "text": "Antibiótico X" }
      }
    }
  ]
}

7. Requisitos Não Funcionais (RNF)RNF.01 - Sincronização: O dashboard de enfermagem deve atualizar via WebSockets sempre que o médico atualizar uma previsão de alta ou um exame for concluído.RNF.02 - Usabilidade: O app de limpeza e enfermaria deve ser operável com apenas uma mão (botões grandes), considerando que o profissional pode estar de luvas ou segurando equipamentos.RNF.03 - Privacidade: Dados de saúde devem seguir as normas da LGPD, garantindo que o acompanhante veja apenas o "fluxo" e não o "prontuário".