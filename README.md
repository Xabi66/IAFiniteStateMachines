# Descrición

Escena con una caseta y un NPC que patrulla alrededor de ella mediante diversos checks. Cada vez que el NPC alcanza un check, da un giro de 360º alrededor.

# Título principal

**Escena caseta**

## Subtítulo

Lista de cambios:

-Añadida una nueva escena con una caseta y un NPC.

-Añadido un nuevo estado **Look** al script *State.cs*. Este estado causa que el NPC realice un giro de 360º. 

-Modificado el estado **Patrol** para que acepte recibir el index del proximo checkpoint mediante el constructor.

-Modificado el estado **Patrol** para que al llegar a un checkpoint cambie al estado **Look** antes de volver a patrullar.

-Se ha desplazado el *SafePointRunawayState* de la escena para que tenga menor tamaño y se ubique dentro de la caseta, de modo que el NPC huya a dentro de la misma.

[Ligazón](https://github.com/Xabi66/IAFiniteStateMachines/blob/main/Assets/Scripts/State.cs)


```csharp
public class Look : State
{
    float rotationSpeed = 360f; //Grados rotados por segundo
    float rotated = 0f; //Rotacion actual         
    int nextCheckpoint= -2; //Proximo checkpoint ao que se dirixirá o NPC
    public Look(GameObject _npc, NavMeshAgent _agent, Animator _animator, Transform _player, int currentIndex)
        : base(_npc, _agent, _animator, _player)
    {
        name = STATE.LOOK;
        nextCheckpoint=currentIndex;
    }

    //=========================================================================
    // Ao entrar no estado, o NPC para
    //=========================================================================
    public override void Enter()
    {
        animator.SetTrigger("isIdle");  // Activa o trigger da animación de inactividade
        agent.isStopped = true;         // Detén o movemento do NPC
        base.Enter();               // Chama ao método Enter da clase base
    }

    //=========================================================================
    // O NPC xira ao redor para comprobar se ve ao xogador
    //=========================================================================
    public override void Update()
    {
        //Se calcula la rotacion en grados para cada frame
        float rotation = rotationSpeed * Time.deltaTime;
        //Se aplica esa rotación al NPC en el eje Y
        npc.transform.Rotate(Vector3.up, rotation);
        //Almacena esa rotación en el total
        rotated += rotation;

        if (CanSeePlayer()) // Se o NPC ve ao xogador...
        {
            nextState = new Pursue(npc, agent, animator, player);  // Crea un estado de persecución
            stage = EVENT.EXIT;                                 // Marca para saír do estado actual
        }

        // Se o NPC xa rematou de xirar, volve a patrullar cara ao seguinte checkpoint
        if (rotated >= 360f)
        { 
            nextState = new Patrol(npc, agent, animator, player, nextCheckpoint);  // Crea un estado de patrulla
            stage = EVENT.EXIT;                                 // Marca para saír do estado actual
        }
    }

    //=========================================================================
    // Ao saír do estado, resetea o trigger da animación
    //=========================================================================
    public override void Exit()
    {
        rotated=0f;                     // Reinicia a cantidade rotada
        animator.ResetTrigger("isIdle");// Limpa o trigger da animación
        agent.isStopped = false;        // Reactiva o movemento do NPC
        base.Exit();                    // Chama ao método Exit da clase base
    }
}

```