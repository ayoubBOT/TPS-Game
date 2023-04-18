using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Animations.Rigging;
using Cinemachine;

public class Player_Movements : MonoBehaviour
{
    CharacterController controller;
    Animator _anim;

    public GameObject handedhandGun, groundedhanGun;
    public RigBuilder rigBuilder;

    [Header("movements vars")]
    [SerializeField] float speed, jumpForce, gravity, timeToJumpApex = 0.4f;

    [Header("var")]
    public Transform Camera;
    public float rotationSpeed = 10f;
    private Quaternion cameraStartRotation;

    [Header("bool")]
    [SerializeField] private bool isGrounded, isJumping, justLanded;

    [Header("Jumping cooldown")]
    public float cooldown = 1.0f;
    public float cooldownTimer = 0.0f;

    Vector3 velocity;

    void Start()
    {
        controller = GetComponent<CharacterController>();
        _anim = GetComponent<Animator>();
        // Calculate gravity and jump velocity
        gravity = -(2 * jumpForce) / Mathf.Pow(timeToJumpApex, 2);
    }

    void Update()
    {
        isGrounded = controller.isGrounded;
        if (velocity.y < 0 && isGrounded)
        {
            velocity.y = -2f;

            _anim.SetBool("jump", false);
        }

        if (!isGrounded)
            _anim.SetBool("Falling", true);
        else
            _anim.SetBool("Falling", false);


        // If the player just landed, trigger the landing animation
        if (justLanded)
        {
            _anim.SetBool("land", true);
            justLanded = false;
        }

        float x = Input.GetAxisRaw("Horizontal");
        float z = Input.GetAxisRaw("Vertical");

        _anim.SetFloat("h", x, .1f, Time.deltaTime);
        _anim.SetFloat("v", z, .1f, Time.deltaTime);


        // Calculate the move direction
        Vector3 cameraForward = Camera.forward;
        Vector3 cameraRight = Camera.right;
        cameraForward.y = 0f;
        cameraRight.y = 0f;
        cameraForward = cameraForward.normalized;
        cameraRight = cameraRight.normalized;
        Vector3 moveDir = cameraForward * z + cameraRight * x;
        moveDir.Normalize();


        Quaternion targetRotation = cameraStartRotation;
        if (Camera != null && Mathf.Abs(Input.GetAxisRaw("Mouse X")) >= 0)
        {
            targetRotation = Quaternion.Euler(0f, Camera.eulerAngles.y, 0f);
        }
        transform.rotation = Quaternion.Lerp(transform.rotation, targetRotation, Time.deltaTime * rotationSpeed);


        controller.Move(moveDir * speed * Time.deltaTime);
        if (moveDir.magnitude > .5f)
        {
            if (Input.GetKeyDown(KeyCode.LeftShift))
            {
                speed = speed * 2;
                _anim.SetBool("run", true);
            }

            if (Input.GetKeyUp(KeyCode.LeftShift))
            {
                speed = speed / 2;
                _anim.SetBool("run", false);
            }
        }

        

        // Check if the player just landed from a jump
        if (isJumping && !justLanded && velocity.y < 0 && isGrounded || !isGrounded)
        {
            justLanded = true;
        }

        if (!isGrounded && cooldownTimer <= 0)
        {
            cooldownTimer = cooldown;
        }

        if (cooldownTimer > 0)
        {
            cooldownTimer -= Time.deltaTime;
        }

        if (Input.GetButtonDown("Jump") && isGrounded && cooldownTimer <= 0)
        {
            isJumping = true;
            jump();
        }

        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);
    }

    void jump()
    {
        _anim.SetBool("jump", true);
        velocity.y = Mathf.Sqrt(jumpForce * -2f * gravity);
    }

    public void ResetJustLanded()
    {
        _anim.SetBool("land", false);
        justLanded = false;
        isJumping = false;
    }

    private void OnControllerColliderHit(ControllerColliderHit hit)
    {
        if (hit.collider.gameObject.tag == "groundedhandGun")
        {
            handedhandGun.SetActive(true);
            groundedhanGun.SetActive(false);

            rigBuilder.enabled = true;
            SwitchToPistolLayer();
        }
    }

    private void SwitchToPistolLayer()
    {
        _anim.SetLayerWeight(1, 1);
        _anim.SetLayerWeight(0, 0);
    }
}

