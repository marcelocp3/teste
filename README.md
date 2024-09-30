
A saida esperada era {'cesto_claro': 2, 'cesto_escuro': 1}. Porém estou recebendo {'cesto_claro': 16, 'cesto_escuro': 4167}. No vídeo, algumas laranjas caem fora do cesto, o que pode estar confundindo o codigo. Atualmente, com os valores das mascaras ajustados, o código esta assim:


import cv2
import numpy as np

class OrangeCounter:
    def __init__(self):
        self.resultado = {"cesto_claro": 0, "cesto_escuro": 0} 
        self.laranjas_anteriores = []

    def detectar_cestos(self, frame):

        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        lower_claro = np.array([0, 111, 185])  
        upper_claro = np.array([19, 149, 255])
        lower_escuro = np.array([0, 126, 58])  
        upper_escuro = np.array([20, 176, 126])

        mask_claro = cv2.inRange(hsv, lower_claro, upper_claro)
        mask_escuro = cv2.inRange(hsv, lower_escuro, upper_escuro)


        contours_claro, _ = cv2.findContours(mask_claro, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        contours_escuro, _ = cv2.findContours(mask_escuro, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        cesto_claro = max(contours_claro, key=cv2.contourArea) if contours_claro else None
        cesto_escuro = max(contours_escuro, key=cv2.contourArea) if contours_escuro else None

        return cesto_claro, cesto_escuro

    def detectar_laranjas(self, frame):
 
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)


        lower_orange = np.array([15, 190, 200])
        upper_orange = np.array([21, 255, 255])


        mask = cv2.inRange(hsv, lower_orange, upper_orange)


        contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)


        return contours

    def laranja_entrou_no_cesto(self, laranja, cesto):
        if cesto is not None:
            x, y, w, h = cv2.boundingRect(cesto)
            laranja_x, laranja_y, laranja_w, laranja_h = cv2.boundingRect(laranja)
            laranja_centro = (laranja_x + laranja_w // 2, laranja_y + laranja_h // 2)

            return x <= laranja_centro[0] <= x + w and y <= laranja_centro[1] <= y + h
        return False

    def run(self, frame):
        cesto_claro, cesto_escuro = self.detectar_cestos(frame)


        laranjas = self.detectar_laranjas(frame)

        novas_laranjas = []
        for laranja in laranjas:
            entrou_claro = self.laranja_entrou_no_cesto(laranja, cesto_claro)
            entrou_escuro = self.laranja_entrou_no_cesto(laranja, cesto_escuro)

            if entrou_claro:
                self.resultado["cesto_claro"] += 1
            elif entrou_escuro:
                self.resultado["cesto_escuro"] += 1
            else:
                novas_laranjas.append(laranja)
        self.laranjas_anteriores = novas_laranjas

        return self.resultado

def main():
    video = cv2.VideoCapture('/home/borg/colcon_ws/ai-marcelocp3/q2/laranja1.mp4') 
    contador = OrangeCounter()

    while True:
        ret, frame = video.read()
        if not ret:
            break
        resultado = contador.run(frame)

        cv2.imshow('Video', frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    video.release()
    cv2.destroyAllWindows()

    print(resultado)

if __name__ == "__main__":
    main()


Quero que voce corrija a saida q esta incorreta no momento, e exiba os contornos das mascaras no video.
