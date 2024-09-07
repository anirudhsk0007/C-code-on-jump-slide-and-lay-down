using UnityEngine;

public class CharacterMovement : MonoBehaviour
{
    public float moveSpeed = 5f;        // Speed of character movement
    public float jumpForce = 7f;        // Force applied for jumping
    public float slideSpeed = 10f;      // Speed during the slide
    public float slideDuration = 1f;    // Duration of the slide
    public float layDownHeight = 0.5f;  // Height of the character when laying down

    private bool isGrounded;             // Check if the character is on the ground
    private bool isSliding;              // Check if the character is sliding
    private bool isLayingDown;           // Check if the character is laying down
    private float originalHeight;        // Original height of the character

    private Rigidbody rb;                // Reference to the Rigidbody component
    private CapsuleCollider col;         // Reference to the CapsuleCollider component

    private void Start()
    {
        // Get the Rigidbody and CapsuleCollider components
        rb = GetComponent<Rigidbody>();
        col = GetComponent<CapsuleCollider>();

        // Store the original height of the character
        originalHeight = col.height;
    }

    private void Update()
    {
        // Get input from the user
        float moveHorizontal = Input.GetAxis("Horizontal"); // A/D or Left/Right arrow keys
        float moveVertical = Input.GetAxis("Vertical");     // W/S or Up/Down arrow keys

        // Create a movement vector
        Vector3 movement = new Vector3(moveHorizontal, 0f, moveVertical);

        // Normalize the movement vector to maintain consistent speed in all directions
        movement = movement.normalized * moveSpeed * Time.deltaTime;

        // Move the character
        transform.Translate(movement, Space.World);

        // Check for jump input
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            Jump();
        }

        // Check for slide input
        if (Input.GetKeyDown(KeyCode.LeftShift) && !isSliding)
        {
            StartCoroutine(Slide());
        }

        // Check for lay down input
        if (Input.GetKeyDown(KeyCode.C))
        {
            LayDown();
        }
        else if (Input.GetKeyUp(KeyCode.C))
        {
            StandUp();
        }
    }

    private void Jump()
    {
        // Apply a vertical force to jump
        rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
        isGrounded = false; // Set grounded state to false after jumping
    }

    private IEnumerator Slide()
    {
        // Set sliding state to true and increase speed
        isSliding = true;
        moveSpeed = slideSpeed;

        // Reduce the height of the character while sliding
        col.height = originalHeight / 2f;

        // Wait for the slide duration
        yield return new WaitForSeconds(slideDuration);

        // Reset the height and speed after sliding
        col.height = originalHeight;
        moveSpeed = 5f;

        // Set sliding state to false
        isSliding = false;
    }

    private void LayDown()
    {
        // Reduce the height of the character when laying down
        col.height = layDownHeight;
        isLayingDown = true;
    }

    private void StandUp()
    {
        // Reset the height when standing up
        col.height = originalHeight;
        isLayingDown = false;
    }

    private void OnCollisionEnter(Collision collision)
    {
        // Check if the character lands on the ground
        if (collision.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }
    }
}
