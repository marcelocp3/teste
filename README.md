import cv2
import numpy as np

class OrangeCounter:
    def __init__(self):
        self.resultado = {"cesto_claro": 0, "cesto_escuro": 0}
        self.laranjas_contadas = []  # Lista para armazenar as laranjas já contadas

    def detectar_cestos(self, frame):
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        # Ajustando os limites para detectar os cestos
        lower_claro = np.array([0, 111, 185])
        upper_claro = np.array([19, 149, 255])
        lower_escuro = np.array([0, 126, 58])
        upper_escuro = np.array([20, 176, 126])

        # Criando máscaras
        mask_claro = cv2.inRange(hsv, lower_claro, upper_claro)
        mask_escuro = cv2.inRange(hsv, lower_escuro, upper_escuro)

        # Encontrando os contornos
        contours_claro, _ = cv2.findContours(mask_claro, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        contours_escuro, _ = cv2.findContours(mask_escuro, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        cesto_claro = max(contours_claro, key=cv2.contourArea) if contours_claro else None
        cesto_escuro = max(contours_escuro, key=cv2.contourArea) if contours_escuro else None

        # Desenhar os contornos dos cestos no frame para visualização
        if cesto_claro is not None:
            cv2.drawContours(frame, [cesto_claro], -1, (255, 255, 0), 2)  # Contorno Azul para cesto claro
        if cesto_escuro is not None:
            cv2.drawContours(frame, [cesto_escuro], -1, (0, 0, 255), 2)  # Contorno Vermelho para cesto escuro

        return cesto_claro, cesto_escuro

    def detectar_laranjas(self, frame):
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        # Ajustando os limites para detectar laranjas
        lower_orange = np.array([15, 190, 200])
        upper_orange = np.array([21, 255, 255])

        # Criando máscara
        mask = cv2.inRange(hsv, lower_orange, upper_orange)

        # Encontrando os contornos
        contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # Desenhar os contornos das laranjas no frame para visualização
        for contour in contours:
            cv2.drawContours(frame, [contour], -1, (0, 255, 0), 2)  # Contorno Verde para laranjas

        return contours

    def laranja_entrou_no_cesto(self, laranja, cesto):
        if cesto is not None:
            x, y, w, h = cv2.boundingRect(cesto)
            laranja_x, laranja_y, laranja_w, laranja_h = cv2.boundingRect(laranja)
            laranja_centro = (laranja_x + laranja_w // 2, laranja_y + laranja_h // 2)

            # Verificar se o centro da laranja está dentro do cesto
            return x <= laranja_centro[0] <= x + w and y <= laranja_centro[1] <= y + h
        return False

    def run(self, frame):
        # Detectar os cestos no frame atual
        cesto_claro, cesto_escuro = self.detectar_cestos(frame)

        # Detectar as laranjas no frame atual
        laranjas = self.detectar_laranjas(frame)

        novas_laranjas = []
        for laranja in laranjas:
            # Verificar se a laranja já foi contada
            if laranja in self.laranjas_contadas:
                continue  # Ignorar a laranja se já foi contada

            entrou_claro = self.laranja_entrou_no_cesto(laranja, cesto_claro)
            entrou_escuro = self.laranja_entrou_no_cesto(laranja, cesto_escuro)

            if entrou_claro:
                self.resultado["cesto_claro"] += 1
                self.laranjas_contadas.append(laranja)  # Marcar laranja como contada
            elif entrou_escuro:
                self.resultado["cesto_escuro"] += 1
                self.laranjas_contadas.append(laranja)  # Marcar laranja como contada
            else:
                novas_laranjas.append(laranja)

        return self.resultado

def main():
    video = cv2.VideoCapture('/home/borg/colcon_ws/ai-marcelocp3/q2/laranja1.mp4') 
    contador = OrangeCounter()

    while True:
        ret, frame = video.read()
        if not ret:
            break

        # Executa a contagem das laranjas e desenha os contornos
        resultado = contador.run(frame)

        # Exibe o frame com os contornos
        cv2.putText(frame, f"Cesto Claro: {resultado['cesto_claro']}", (10, 50),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)
        cv2.putText(frame, f"Cesto Escuro: {resultado['cesto_escuro']}", (10, 100),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)

        cv2.imshow('Video', frame)

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    video.release()
    cv2.destroyAllWindows()

    # Imprime o resultado final
    print(resultado)

if __name__ == "__main__":
    main()