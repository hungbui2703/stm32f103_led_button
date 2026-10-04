/* Register Definitions */
#define RCC_APB2ENR   (*(volatile unsigned int*)0x40021018)

#define GPIOA_CRL     (*(volatile unsigned int*)0x40010800)
#define GPIOA_IDR     (*(volatile unsigned int*)0x40010808)
#define GPIOA_ODR     (*(volatile unsigned int*)0x4001080C)

#define GPIOB_CRL     (*(volatile unsigned int*)0x40010C00)
#define GPIOB_IDR     (*(volatile unsigned int*)0x40010C08)
#define GPIOB_ODR     (*(volatile unsigned int*)0x40010C0C)

#define GPIOC_CRH     (*(volatile unsigned int*)0x40011004)
#define GPIOC_ODR     (*(volatile unsigned int*)0x4001100C)

#define AFIO_MAPR     (*(volatile unsigned int*)0x40010004)

/* Simple Delay Function */
void delay(volatile unsigned int time) {
    while(time--);
}

int main(void) {
    // 1. Enable Clocks: AFIO (bit 0), GPIOA (bit 2), GPIOB (bit 3), GPIOC (bit 4)
    RCC_APB2ENR |= (1 << 0) | (1 << 2) | (1 << 3) | (1 << 4);

    // 2. Disable JTAG to use PB3
    AFIO_MAPR |= (1 << 25);

    // 3. Pin Config:
    // PA1=Out (LED2), PA2=In (Btn Right), PB0=In (Btn Left), PB3=Out (LED3), PC13=Out (LED1)
    GPIOA_CRL &= ~((0xF << 4) | (0xF << 8));
    GPIOA_CRL |=  ((0x2 << 4) | (0x8 << 8));

    GPIOB_CRL &= ~((0xF << 0) | (0xF << 12));
    GPIOB_CRL |=  ((0x8 << 0) | (0x2 << 12));

    GPIOC_CRH &= ~(0xF << 20);
    GPIOC_CRH |=  (0x2 << 20);

    // 4. Enable Pull-ups for Inputs (Setting ODR bit for input pins)
    GPIOB_ODR |= (1 << 0); // PB0 Pull-up
    GPIOA_ODR |= (1 << 2); // PA2 Pull-up

    while (1) {
            // --- SWAPPED BUTTON ASSIGNMENTS ---
            // Now 'left' reads from PA2 and 'right' reads from PB0
            int left  = !(GPIOA_IDR & (1 << 2)); // Now Button 2 (Right side) acts as Left
            int right = !(GPIOB_IDR & (1 << 0)); // Now Button 1 (Left side) acts as Right

            if (left && right) {
                // BOTH PRESSED: All LEDs ON
                GPIOC_ODR &= ~(1 << 13); // LED 1 ON (Low)
                GPIOA_ODR |= (1 << 1);   // LED 2 ON (High)
                GPIOB_ODR |= (1 << 3);   // LED 3 ON (High)
            }
            else if (left) {
                // LEFT logic
                // LED 1 toggles (1s), LED 2 ON, LED 3 OFF
                GPIOA_ODR |= (1 << 1);
                GPIOB_ODR &= ~(1 << 3);
                GPIOC_ODR ^= (1 << 13);
                delay(1000000);
            }
            else if (right) {
                // RIGHT logic
                // LED 1 OFF, LED 3 ON, LED 2 OFF
                GPIOC_ODR |= (1 << 13); // LED 1 OFF (High)
                GPIOB_ODR |= (1 << 3);  // LED 3 ON
                GPIOA_ODR &= ~(1 << 1); // LED 2 OFF
            }
            else {
                // IDLE: LED 1 toggles (3s), LED 2 & 3 OFF
                GPIOA_ODR &= ~(1 << 1);
                GPIOB_ODR &= ~(1 << 3);
                GPIOC_ODR ^= (1 << 13);
                delay(3000000);
            }
        }
}
