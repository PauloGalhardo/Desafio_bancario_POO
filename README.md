# Desafio_bancario_POO

from abc import ABC, abstractclassmethod, abstractproperty
from datetime import datetime

operacao = """
            # o uso de numeros aos inves de letras, foi pensando em acessar este programa via ligação telefônica (consulta de extrato/saldo)
[1] Deposito
[2] Saque
[3] Extrato
[4] Novo Cliente
[5] Nova Conta
[6] Listar Contas
[9] Sair

=> Informe o número da operação desejada: """

# saldo = 0
transacoes = []
saques_diarios = 0
limite_saques = 3
limite_valor_saque = 500


class Cliente:
    def __init__(self, endereco):
        self.endereco = endereco
        self.contas = []

    def transacao(self, conta, transacao):
        transacao.registrar(conta)

    def informar_conta(self, conta):
        self.contas.append(conta)


class DadosClientes(Cliente):
    def __init__(self, nome, data_nascimento, cpf, endereco):
        super().__init__(endereco)
        self.nome = nome
        self.data_nascimento = data_nascimento
        self.cpf = cpf


class Conta:
    def __init__(self, numero, cliente):
        self._saldo = 0
        self._numero = numero
        self._agencia = "0001"
        self._cliente = cliente
        self._historico = Historico()

    def sacar(self, valor):
        """Método genérico de saque."""
        if valor > self._saldo:
            print("\n@@@ Operação falhou! Saldo insuficiente. @@@")
            return False

        self._saldo -= valor
        self._historico.adicionar_transacao(Saque(valor))
        print(f"\nSaque de R$ {valor:.2f} realizado com sucesso!")
        return True


    @classmethod
    def nova_conta(cls, cliente, numero):
        return cls(numero, cliente)

    @property
    def saldo(self):
        return self._saldo

    @property
    def numero(self):
        return self._numero

    @property
    def agencia(self):
        return self._agencia

    @property
    def cliente(self):
        return self._cliente

    @property
    def historico(self):
        return self._historico


def depositar(saldo, valor, extrato, /):
    if valor > 0:
        saldo += valor
        mensagem = f"Depósito: + R$ {valor:.2f}\n"
        extrato += mensagem
        transacoes.append(mensagem)
        print(f"Depósito de R$ {valor:.2f} realizado com sucesso!")
    else:
        print("O valor do depósito deve ser positivo.")
    return saldo, extrato


class ContaCorrente(Conta):
    def __init__(self, numero, cliente, limite=500, limite_saques=3):
        super().__init__(numero, cliente)
        self.limite = limite
        self.limite_saques = limite_saques

    def sacar(self, valor):
        """Método de saque com validação do limite e número máximo de saques."""
        numero_saques = len(
            [transacao for transacao in self.historico.transacoes if transacao["tipo"] == "Saque"]
        )

        if valor > self.limite:
            print("\n@@@ Operação falhou! O valor do saque excede o limite. @@@")
            return False

        if numero_saques >= self.limite_saques:
            print("\n@@@ Operação falhou! Número máximo de saques excedido. @@@")
            return False

        return super().sacar(valor)
    
    def __str__(self):
        return f"""\
            Agência:\t{self.agencia}
            C/C:\t\t{self.numero}
            Titular:\t{self.cliente.nome}
        """


class Historico:
    def __init__(self):
        self._transacoes = []

    @property
    def transacoes(self):
        return self._transacoes

    def adicionar_transacao(self, transacao):
        self._transacoes.append(
            {
                "tipo": transacao.__class__.__name__,
                "valor": transacao.valor,
                "data": datetime.now().strftime("%d-%m-%Y %H:%M:%S"),
            }
        )


class Transacao(ABC):
    @property
    @abstractproperty
    def valor(self):
        pass

    @abstractclassmethod
    def registrar(self, conta):
        pass


class Saque(Transacao):
    def __init__(self, valor):
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        sucesso_transacao = conta.sacar(self.valor)

        if sucesso_transacao:
            conta.historico.adicionar_transacao(self)


class Deposito(Transacao):
    def __init__(self, valor):
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        sucesso_transacao = conta.depositar(self.valor)

        if sucesso_transacao:
            conta.historico.adicionar_transacao(self)


def sacar(*, saldo, valor, extrato, limite_valor_saque, saques_diarios, limite_saques):
    if saques_diarios >= limite_saques:
        print("Limite de saques diários atingido!")
    elif valor > limite_valor_saque:
        print(
            f"Valor de saque excede o limite de R$ {limite_valor_saque:.2f}!")
    elif valor > saldo:
        print("Saldo insuficiente para realizar o saque.")
    else:
        saldo -= valor
        mensagem = f"Saque: - R$ {valor:.2f}\n"
        extrato += mensagem
        transacoes.append(mensagem)
        saques_diarios += 1

        print(f"Saque de R$ {valor:.2f} realizado com sucesso!")

    return saldo, extrato, saques_diarios


def exibir_extrato(saldo, /, *, extrato):
    print("\n========= EXTRATO ==========")
    if not transacoes:
        print("Nenhuma transação realizada.")
    else:
        for transacao in transacoes:
            print(transacao)
            print(f"Saldo atual: R$ {saldo:.2f}")
            print("=================\n")


def criar_cliente(clientes):
    cpf = input("Informe o CPF (so numeros): ")
    cliente = filtrar_cliente(cpf, clientes)

    if cliente:
        print("CPF já cadastrado, informe CPF não cadastrado")
        return

    nome = input("Informe o nome completo: ")
    data_nascimento = input("Informe a data do seu nascimento (dd-mm-aaaa): ")
    endereco = input(
        "Informe seu endereço (logradouro, numero - bairro - cidade/estado): ")

    clientes.append({"nome": nome, "data_nascimento": data_nascimento,
                    "cpf": cpf, "endereco": endereco})

    print("Cadastro efetuado com sucesso")


def filtrar_cliente(cpf, clientes):
    clientes_filtrados = [
        cliente for cliente in clientes if cliente["cpf"] == cpf]
    return clientes_filtrados[0] if clientes_filtrados else None


def criar_conta(agencia, numero_conta, clientes):
    cpf = input("Informe o CPF do cliente: ")
    cliente = filtrar_cliente(cpf, clientes)

    if cliente:
        print("Conta criada com sucesso!")
        return {"agencia": agencia, "numero_conta": numero_conta, "cliente": cliente}

    print("Cliente não encontrado.")
    return None


def listar_contas(contas):
    resultado = ""
    for conta in contas:
        linha = f"""\n==============================================
        Agencia: {conta['agencia']}
        Conta: {conta['numero_conta']}
        Titular: {conta['cliente']['nome']}
        =============================="""
        resultado += linha
        print(resultado)
    return resultado


def main():
    limite_saques = 3
    agencia = "0001"

    saldo = 0
    limite = 500
    extrato = ""
    saques_diarios = 0
    clientes = []
    contas = []

    while True:
        opcao = input(operacao)

        if opcao == "1":
            valor = float(input("Informe valor do depósito: "))

            saldo, extrato = depositar(saldo, valor, extrato)

        elif opcao == "2":
            valor = float(input("Informe valor do saque: "))

            saldo, extrato, saques_diarios = sacar(
                saldo=saldo,
                valor=valor,
                extrato=extrato,
                limite_valor_saque=limite_valor_saque,
                saques_diarios=saques_diarios,
                limite_saques=limite_saques,
            )

        elif opcao == "3":
            exibir_extrato(saldo, extrato=extrato)

        elif opcao == "4":
            criar_cliente(clientes)

        elif opcao == "5":
            numero_conta = len(contas) + 1
            conta = criar_conta(agencia, numero_conta, clientes)

            if conta:
                contas.append(conta)

        elif opcao == "6":
            listar_contas(contas)

        elif opcao == "9":
            break

        else:
            print(
                "Operação inválida, selecione novamente a operação desejada. ")


main()
