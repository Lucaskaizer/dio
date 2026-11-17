Simulação segura de comportamentos de malware
Ransomware simulado • Keylogger simulado • Defesa e prevenção
Tecnologia: Python | Ambiente: laboratório local ou máquina virtual
 
Resumo executivo
Este estudo documenta dois comportamentos associados a malwares — ransomware e keylogger — por meio de simulações didáticas em Python. O objetivo é compreender seu funcionamento geral, os riscos à confidencialidade, integridade e disponibilidade dos dados e as medidas de prevenção.
Os protótipos são deliberadamente seguros: o simulador de ransomware atua somente sobre um arquivo artificial criado para o exercício, permite recuperação, não apaga originais e não percorre diretórios. O simulador de keylogger registra apenas texto digitado voluntariamente no próprio terminal; não captura teclas globais, não opera oculto e não transmite dados por e-mail. Assim, demonstra o conceito sem produzir ferramenta de vigilância ou exfiltração.
1. Descrição do desafio
O desafio solicita implementar, documentar e compartilhar um projeto prático em Python para simular o comportamento de malwares em ambiente seguro. Os tópicos apresentados são:
•	Ransomware simulado: criar arquivos de teste, implementar criptografia e descriptografia e gerar mensagem de “resgate”.
•	Keylogger simulado: demonstrar registro de entradas em arquivo .txt; o enunciado também menciona furtividade e envio automático por e-mail.
•	Reflexão sobre defesa: documentar prevenção e defesa, incluindo antivírus, firewall, sandboxing e conscientização do usuário.
O enunciado informa que o desafio é flexível: pode-se implementar os exemplos ou documentar em detalhes estudos, reflexões e aprendizados. Este relatório segue a opção de documentação detalhada, com protótipos locais seguros.
2. Objetivos
•	Compreender o conceito e o impacto de ransomware e keylogger.
•	Demonstrar criptografia e recuperação de um arquivo de teste.
•	Simular registro de texto digitado voluntariamente na própria aplicação.
•	Identificar os riscos de ocultação, persistência e exfiltração.
•	Relacionar controles técnicos e comportamentais à prevenção de incidentes.
•	Documentar procedimentos, resultados esperados, limitações e aprendizados.
3. Ambiente e preparação
Executar em uma pasta exclusiva de laboratório, preferencialmente em máquina virtual sem dados pessoais. Criar snapshot antes dos testes e manter cópia dos arquivos originais.
Componente	Finalidade
Python 3	Linguagem dos protótipos
Editor ou IDE	VS Code, IDLE ou similar
Pasta de laboratório	Armazenar scripts e arquivos artificiais
cryptography	Biblioteca usada no exemplo Fernet
Máquina virtual (opcional)	Isolamento e restauração do ambiente
Instalação da dependência:
python -m pip install cryptography
Estrutura sugerida:
estudo_malware/
├── ransomware_simulado.py
├── keylogger_simulado.py
├── laboratorio/
│   ├── arquivo_teste.txt
│   ├── arquivo_teste.txt.encrypted
│   ├── chave.key
│   └── aviso_resgate.txt
└── registros/
4. Estudo 1 — Ransomware simulado
4.1 Conceito e funcionamento
Ransomware é uma categoria de malware que impede ou dificulta o acesso a dados, frequentemente por criptografia, e exige pagamento para supostamente devolver o acesso. Incidentes reais podem também envolver roubo de informações, ameaça de divulgação, propagação e interrupção de operações.
Na simulação didática, o foco é observar a transformação de um arquivo legível em conteúdo cifrado e sua recuperação com uma chave. Não é necessário reproduzir técnicas perigosas de um malware real para compreender o conceito.
4.2 Limites de segurança do protótipo
•	Cria e manipula somente um arquivo artificial dentro da pasta laboratorio.
•	Preserva o arquivo original e gera uma cópia cifrada.
•	Gera uma chave local para recuperação.
•	Não percorre pastas, unidades ou compartilhamentos.
•	Não apaga arquivos, não se propaga, não persiste e não inicia automaticamente.
4.3 Código demonstrativo
Salvar como ransomware_simulado.py. O exemplo usa Fernet, que oferece criptografia autenticada:
from pathlib import Path
from cryptography.fernet import Fernet

BASE = Path(__file__).resolve().parent
LAB = BASE / "laboratorio"
ORIGINAL = LAB / "arquivo_teste.txt"
CIFRADO = LAB / "arquivo_teste.txt.encrypted"
CHAVE = LAB / "chave.key"
AVISO = LAB / "aviso_resgate.txt"

def preparar():
    LAB.mkdir(exist_ok=True)
    if not ORIGINAL.exists():
        ORIGINAL.write_text("ARQUIVO FICTICIO PARA TESTE.\\n", encoding="utf-8")

def criptografar():
    if not ORIGINAL.exists():
        raise FileNotFoundError("Arquivo de teste ausente.")
    if CIFRADO.exists():
        raise FileExistsError("Já existe cópia cifrada.")
    chave = Fernet.generate_key()
    f = Fernet(chave)
    CIFRADO.write_bytes(f.encrypt(ORIGINAL.read_bytes()))
    CHAVE.write_bytes(chave)
    AVISO.write_text(
        "SIMULACAO DIDATICA: arquivo de teste criptografado.\\n"
        "Use a opção de recuperação do próprio laboratório.\\n",
        encoding="utf-8"
    )
    print("Concluído. O original foi preservado.")

def recuperar():
    if not CHAVE.exists() or not CIFRADO.exists():
        raise FileNotFoundError("Chave ou arquivo cifrado ausente.")
    f = Fernet(CHAVE.read_bytes())
    dados = f.decrypt(CIFRADO.read_bytes())
    (LAB / "arquivo_teste_recuperado.txt").write_bytes(dados)
    print("Recuperado em arquivo_teste_recuperado.txt")

if __name__ == "__main__":
    preparar()
    print("1 - Criptografar arquivo fictício")
    print("2 - Recuperar arquivo")
    op = input("Escolha: ").strip()
    if op == "1": criptografar()
    elif op == "2": recuperar()
    else: print("Opção inválida.")
4.4 Execução e validação
•	Salvar o script e instalar cryptography.
•	Executar: python ransomware_simulado.py.
•	Selecionar 1 e verificar a criação do arquivo cifrado, chave e aviso.
•	Confirmar que arquivo_teste.txt continua preservado.
•	Executar novamente e selecionar 2.
•	Comparar o conteúdo do original com arquivo_teste_recuperado.txt.
4.5 Resultado esperado e análise
O arquivo .encrypted não deve ser legível como texto comum. A recuperação com a chave correta deve produzir conteúdo idêntico ao original. Se a chave estiver ausente ou incorreta, a recuperação falha. Isso demonstra a importância da proteção de chaves e de backups independentes.
Limitação: o protótipo não reproduz propagação, persistência, evasão, exclusão de backups, criptografia em massa ou comunicação com operador criminoso. Esses recursos foram excluídos intencionalmente.
5. Estudo 2 — Keylogger simulado
5.1 Conceito e riscos
Keylogger é um mecanismo que registra entradas do teclado. Em contexto malicioso, pode capturar credenciais, mensagens, dados financeiros e informações pessoais. A coleta oculta e o envio remoto de registros representam risco à privacidade e podem comprometer contas.
O desafio menciona gravação em .txt, furtividade e envio automático por e-mail. Para manter a demonstração segura, o protótipo registra somente o texto que o participante digita voluntariamente no próprio terminal. Não monitora o teclado global, não fica oculto e não envia dados por e-mail.
5.2 Código demonstrativo seguro
Salvar como keylogger_simulado.py:
from pathlib import Path
from datetime import datetime

BASE = Path(__file__).resolve().parent
PASTA = BASE / "registros"
LOG = PASTA / "entrada_manual.txt"

def demonstrar():
    PASTA.mkdir(exist_ok=True)
    print("Digite somente texto fictício neste prompt.")
    print("Digite /sair para encerrar.")
    with LOG.open("a", encoding="utf-8") as arquivo:
        while True:
            texto = input("Texto de teste: ")
            if texto.strip().lower() == "/sair":
                break
            hora = datetime.now().isoformat(timespec="seconds")
            arquivo.write(f"[{hora}] {texto}\\n")
    print(f"Registro local salvo em: {LOG}")

if __name__ == "__main__":
    demonstrar()
5.3 Execução e validação
•	Executar: python keylogger_simulado.py.
•	Digitar apenas frases fictícias no prompt.
•	Digitar /sair para encerrar.
•	Abrir registros/entrada_manual.txt e verificar texto e data/hora.
•	Remover o arquivo de teste quando não for mais necessário.
5.4 Furtividade e envio por e-mail — análise defensiva
Malwares podem tentar ocultar janelas, utilizar nomes enganosos, iniciar junto com o sistema, manter persistência e evitar ferramentas de segurança. O envio de logs por e-mail é uma forma de exfiltração: dados coletados localmente são transmitidos para fora do dispositivo.
Essas funções não são implementadas neste projeto. O estudo aborda como reconhecê-las: revisar programas iniciados com o sistema, investigar processos desconhecidos, permissões excessivas, conexões de saída inesperadas e uso anormal de serviços de e-mail. Uma demonstração de comunicação, se necessária, deve utilizar dados sintéticos e servidor local de teste, nunca teclas ou dados pessoais.
6. Reflexão sobre defesa e prevenção
Controle	Como ajuda	Recomendação
Antivírus/EDR	Detecta padrões e comportamentos suspeitos.	Manter proteção ativa, atualizada e alertas habilitados.
Firewall	Controla conexões de entrada e saída.	Restringir tráfego e bloquear serviços desnecessários.
Sandboxing	Isola arquivos e comportamentos suspeitos.	Analisar em ambiente descartável, sem dados reais.
Backups	Permitem recuperar dados.	Manter cópias offline/imutáveis e testar restauração.
Atualizações	Corrigem vulnerabilidades.	Atualizar sistema, navegador e aplicações.
Menor privilégio	Limita impacto de processo comprometido.	Evitar uso diário de conta administrativa.
MFA e senhas	Reduzem comprometimento de contas.	Usar senhas únicas, gerenciador e MFA.
Conscientização	Reduz riscos de phishing e anexos maliciosos.	Treinar usuários para identificar e reportar suspeitas.
7. Resposta a incidente
•	Isolar o equipamento suspeito da rede conforme orientação da equipe responsável.
•	Comunicar imediatamente a equipe de TI/segurança.
•	Preservar evidências, horários, alertas e nomes de arquivos.
•	Não executar novamente o arquivo suspeito nem apagar evidências.
•	Avaliar sistemas e contas afetados.
•	Restaurar dados de backup confiável após validar o ambiente.
•	Redefinir credenciais comprometidas e revisar acessos.
8. Resultados e aprendizados
O simulador de ransomware permite visualizar a diferença entre conteúdo original e conteúdo cifrado, além de demonstrar que a recuperação depende da chave correta. Preservar o original permite comparar o antes e o depois sem risco de perda de dados.
O simulador de entrada manual demonstra como texto digitado pode ser armazenado em um arquivo de log. Mesmo uma demonstração simples evidencia que logs podem conter informações sensíveis e precisam ser protegidos. A captura global e o envio remoto aumentariam o risco de abuso, por isso foram substituídos por entrada voluntária e transparente.
O aprendizado central é que a segurança depende de camadas: atualização, proteção de endpoint, firewall, menor privilégio, backup testado, autenticação forte, monitoramento e conscientização.
10. Conclusão
O estudo permitiu compreender, por simulações controladas em Python, dois comportamentos associados a malwares: a criptografia de arquivos característica de ransomware e o registro de entradas associado a keyloggers. O projeto foi delimitado para não causar danos, não coletar dados de terceiros e não transmitir informações.
A simulação evidencia que a criptografia pode tornar arquivos inacessíveis sem a chave adequada e que registros de entrada podem expor informações sensíveis. Como medidas de defesa, destacam-se backups offline/imutáveis, proteção de endpoint, firewall, atualizações, autenticação multifator, menor privilégio, monitoramento e conscientização.
Conclui-se que estudar o comportamento de ameaças em ambiente controlado ajuda a reconhecer sinais de comprometimento e planejar medidas preventivas. A execução segura e a documentação transparente são tão importantes quanto o código.

