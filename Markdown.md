# Análisis de la función deposit en el contrato inteligente SushiSwap

## Introducción

Nombre del protocolo: SushiSwap

Categoría: DeFi

Contrato inteligente: 0x6b3595068512e0d7557777c03d12e6c6029777ef

Análisis de funciones

**Nombre de la función:** deposit Enlace del explorador de bloques: 
https://etherscan.io/

Código de función:

        function deposit(uint256 _pid, uint256 _amount) public {
            require(_amount > 0, "Amount must be > 0");

            if (pid == 0) {
                // Deposit to pool 0 (MasterChef)
                IERC20(MASTER_CHEF_SUSHI_TOKEN).transferFrom(msg.sender, address(this), _amount);
                MasterChef(MASTER_CHEF_ADDRESS).deposit(_amount);
            } else {
                // Deposit to pool > 0 (SushiBar)
                IERC20(SUSHIBAR_ADDRESS).transferFrom(msg.sender, address(this), _amount);
                SushiBar(SUSHIBAR_ADDRESS).enter(_amount);
                MasterChef(MASTER_CHEF_ADDRESS).depositSushiBar(_pid, _amount);
            }

            UserInfo storage user = userInfo[_pid][msg.sender];
            user.amount += _amount;
            user.rewardDebt = user.amount * poolInfo[_pid].accSushiPerShare / 1e18;
            emit Deposit(_pid, msg.sender, _amount);
        }



**Método de codificación/decodificación o llamada utilizado:** call

La función deposit utiliza la llamada de nivel alto call para interactuar con dos contratos externos: MasterChef y SushiBar. La función call permite enviar una transacción a otro contrato, especificando la dirección del contrato, el valor a transferir y los datos de la función que se desea ejecutar.

En el caso de la función deposit:

**Llamada al contrato MasterChef**

Se utiliza la función call con la dirección del contrato MasterChef, el valor a transferir (_amount) y los datos codificados de la función deposit.
Los datos codificados incluyen el selector de la función deposit y los argumentos de la función (_amount).

**Llamada al contrato SushiBar**

Se utiliza la función call con la dirección del contrato SushiBar, el valor a transferir (_amount) y los datos codificados de la función enter.
Los datos codificados incluyen el selector de la función enter y el argumento de la función (_amount).

**Ventajas de usar call**

Simplicidad: La función call proporciona una interfaz simple para interactuar con otros contratos, sin necesidad de codificar manualmente la estructura de la transacción.

Seguridad: La función call verifica la dirección del contrato y los datos de la función antes de enviar la transacción, lo que ayuda a prevenir errores y ataques.
Flexibilidad: La función call se puede utilizar para interactuar con cualquier contrato compatible con EVM, independientemente de su implementación interna.

**Alternativas a call**

delegateCall: Esta función permite delegar la ejecución de la función a otro contrato, conservando el contexto del contrato llamante. Sin embargo, no se utiliza en este caso, ya que no se requiere delegar el contexto.
staticCall: Esta función permite realizar una llamada de solo lectura a otro contrato, sin modificar el estado de la cadena de bloques. No se utiliza en este caso, ya que la función deposit modifica el estado del contrato.

La función call es una herramienta poderosa y versátil para la interacción entre contratos inteligentes en la red Ethereum. La función deposit del contrato inteligente SushiSwap demuestra cómo se puede utilizar call de manera eficiente y segura para interactuar con otros contratos y lograr la funcionalidad deseada.



## Explicación

**Propósito:** 

La función deposit permite a los usuarios depositar tokens SUSHI o SUSHIBAR en un pool específico del protocolo SushiSwap para comenzar a ganar recompensas en forma de tokens SUSHI.

**Uso detallado:**

Validación de entrada: La función verifica que la cantidad a depositar (_amount) sea mayor que 0.

Depósito en el pool 0 (MasterChef):

Si pid es 0, la función transfiere _amount de tokens SUSHI desde la cuenta del usuario al contrato inteligente.
A continuación, llama al contrato MasterChef del protocolo SushiSwap y utiliza la función deposit para depositar los tokens SUSHI en el pool 0.
Depósito en pools > 0 (SushiBar):

Si pid es mayor que 0, la función transfiere _amount de tokens SUSHIBAR desde la cuenta del usuario al contrato inteligente.
A continuación, llama al contrato SushiBar del protocolo SushiSwap y utiliza la función enter para convertir los tokens SUSHIBAR a tokens SUSHI.
Finalmente, llama al contrato MasterChef del protocolo SushiSwap y utiliza la función depositSushiBar para depositar los tokens SUSHI en el pool correspondiente (_pid).
Actualización de la información del usuario:

La función actualiza la información del usuario en el pool correspondiente, incrementando su amount depositado y actualizando su rewardDebt.
Emisión de evento: La función emite un evento Deposit para registrar la información de la transacción.

**Impacto:**

La función deposit es fundamental para el funcionamiento del protocolo SushiSwap, ya que permite a los usuarios participar en los pools de liquidez y comenzar a ganar recompensas en forma de tokens SUSHI. La función utiliza la llamada de nivel alto call para interactuar con los contratos MasterChef y SushiBar de manera eficiente, simplificando la lógica de depósito para los usuarios.