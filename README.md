using UnityEngine;

public class AimbotExample : MonoBehaviour
{
    public Transform target; // El enemigo al que apuntar
    public float aimSpeed = 5f;

    void Update()
    {
        if (target != null)
        {
            Vector3 direction = (target.position - transform.position).normalized;
            Quaternion lookRotation = Quaternion.LookRotation(direction);
            transform.rotation = Quaternion.Slerp(transform.rotation, lookRotation, aimSpeed * Time.deltaTime);
        }
    }
}5152624422}8830252767
