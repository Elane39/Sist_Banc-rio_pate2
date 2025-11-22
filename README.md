from datetime import datetime

# ---------------- CONSTANTES ---------------- #
AGENCIA = "0001"

# ---------------- BASE DE DADOS ---------------- #
usuarios = []
contas = []

# ---------------- FUNÇÕES DE USUÁRIO ---------------- #

def criar_usuario():
    print("\n=== CRIAR USUÁRIO ===")
    cpf = input("CPF (somente números): ")

    # Verificar se CPF já existe
    for usuario in usuarios:
        if usuario["cpf"] == cpf:
            print("❌ ERRO: Já existe um usuário com este CPF!")
            return
    
    nome = input("Nome Completo: ")
    data_nasc = input("Data de nascimento (dd/mm/aaaa): ")
    endereco = input("Endereço (Rua, Nº - Cidade/UF): ")

    usuario = {
        "nome": nome,
        "data_nascimento": data_nasc,
        "cpf": cpf,
        "endereco": endereco,
        "contas": []
    }

    usuarios.append(usuario)
    print("✔ Usuário criado com sucesso!")


# ---------------- FUNÇÕES DE CONTA ---------------- #

def filtrar_usuario_por_cpf(cpf):
    for usuario in usuarios:
        if usuario["cpf"] == cpf:
            return usuario
    return None


def criar_conta():
    print("\n=== CRIAR CONTA CORRENTE ===")
    cpf = input("Informe o CPF do usuário: ")

    usuario = filtrar_usuario_por_cpf(cpf)
    if not usuario:
        print("❌ Usuário não encontrado! Cadastre o usuário primeiro.")
        return
    
    numero_conta = len(contas) + 1

    conta = {
        "agencia": AGENCIA,
        "numero": numero_conta,
        "usuario": usuario,
        "saldo": 0.0,
        "extrato": [],
        "saques_dia": 0,
        "ultima_data_saque": None
    }

    contas.append(conta)
    usuario["contas"].append(conta)

    print(f"✔ Conta criada! Agência: {AGENCIA} | Conta: {numero_conta}")


def selecionar_conta():
    cpf = input("Informe o CPF: ")
    usuario = filtrar_usuario_por_cpf(cpf)

    if not usuario:
        print("❌ Usuário não encontrado.")
        return None
    
    if not usuario["contas"]:
        print("❌ Este usuário não possui contas.")
        return None
    
    print("\nContas disponíveis:")
    for conta in usuario["contas"]:
        print(f"→ Conta {conta['numero']} (Agência {conta['agencia']})")

    numero = int(input("Digite o número da conta: "))

    for conta in usuario["contas"]:
        if conta["numero"] == numero:
            return conta
    
    print("❌ Conta não encontrada.")
    return None


# ---------------- OPERAÇÕES BANCÁRIAS ---------------- #

# DEPÓSITO — positional only
def depositar(saldo, valor, extrato, /):
    if valor <= 0:
        print("❌ Valor inválido. Informe um valor positivo.")
        return saldo, extrato
    
    saldo += valor
    extrato.append(("Depósito", valor))
    print(f"✔ Depósito de R$ {valor:.2f} realizado!")
    return saldo, extrato


# SAQUE — keyword-only
def sacar(*, saldo, valor, extrato, conta):
    hoje = datetime.now().date()

    # Reset diário
    if conta["ultima_data_saque"] != hoje:
        conta["saques_dia"] = 0
        conta["ultima_data_saque"] = hoje

    if valor > saldo:
        print("❌ Saldo insuficiente.")
        return saldo, extrato
    
    if valor > 1500:
        print("❌ Saque máximo permitido: R$ 1500.00")
        return saldo, extrato

    if conta["saques_dia"] >= 3:
        print("❌ Limite de 3 saques diários atingido.")
        return saldo, extrato

    saldo -= valor
    extrato.append(("Saque", -valor))
    conta["saques_dia"] += 1

    print(f"✔ Saque de R$ {valor:.2f} realizado!")
    return saldo, extrato


# EXTRATO — posição + nomeado
def exibir_extrato(saldo, /, *, extrato):
    print("\n====== EXTRATO ======")

    if not extrato:
        print("Não há movimentações.")
    else:
        for tipo, valor in extrato:
            print(f"{tipo}: R$ {valor:.2f}")

    print(f"\nSaldo atual: R$ {saldo:.2f}")
    print("=====================\n")


# ---------------- MENU PRINCIPAL ---------------- #

def menu():
    while True:
        print("""
======= SISTEMA BANCÁRIO =======

[1] Criar Usuário
[2] Criar Conta Corrente
[3] Depositar
[4] Sacar
[5] Extrato
[6] Listar contas
[0] Sair

================================
""")

        opcao = input("Escolha uma opção: ")

        if opcao == "1":
            criar_usuario()

        elif opcao == "2":
            criar_conta()

        elif opcao == "3":
            conta = selecionar_conta()
            if conta:
                valor = float(input("Valor do depósito: R$ "))
                conta["saldo"], conta["extrato"] = depositar(conta["saldo"], valor, conta["extrato"])

        elif opcao == "4":
            conta = selecionar_conta()
            if conta:
                valor = float(input("Valor do saque: R$ "))
                conta["saldo"], conta["extrato"] = sacar(saldo=conta["saldo"], valor=valor, extrato=conta["extrato"], conta=conta)

        elif opcao == "5":
            conta = selecionar_conta()
            if conta:
                exibir_extrato(conta["saldo"], extrato=conta["extrato"])

        elif opcao == "6":
            print("\n=== LISTA DE CONTAS ===")
            for conta in contas:
                print(f"Agência: {conta['agencia']} | Conta: {conta['numero']} | CPF: {conta['usuario']['cpf']}")
            print("=======================\n")

        elif opcao == "0":
            print("Saindo... Até logo!")
            break

        else:
            print("❌ Opção inválida.")

# Iniciar o sistema
menu()

