# teste

import cv2

import cv2
import numpy as np

class OrangeCounter:
    def __init__(self):
        # Inicializamos o dicionário para contar as laranjas
        self.resultado = {"cesto_claro": 0, "cesto_escuro": 0}  # l4r4nj4
        # Definimos áreas aproximadas para os dois cestos (isso depende do vídeo)
        self.cesto_claro_area = [(100, 300), (400, 500)]  # Coordenadas do cesto claro
        self.cesto_escuro_area = [(500, 300), (800, 500)]  # Coordenadas do cesto escuro

    def run(self, frame):
        # Convertendo a imagem para o espaço de cor HSV
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        # Definindo o intervalo de cor para detectar a cor laranja
        lower_orange = np.array([5, 100, 100])
        upper_orange = np.array([15, 255, 255])

        # Criando uma máscara para detectar a cor laranja
        mask = cv2.inRange(hsv, lower_orange, upper_orange)

        # Encontrando contornos na máscara para identificar objetos laranjas
        contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # Verificamos se algum contorno (laranja) está entrando nos cestos
        for contour in contours:
            # Obtemos o ponto central do objeto laranja detectado
            x, y, w, h = cv2.boundingRect(contour)
            center = (x + w // 2, y + h // 2)

            # Verificamos se o centro da laranja está em algum dos cestos
            if self.cesto_claro_area[0][0] <= center[0] <= self.cesto_claro_area[1][0] and \
               self.cesto_claro_area[0][1] <= center[1] <= self.cesto_claro_area[1][1]:
                self.resultado["cesto_claro"] += 1
            elif self.cesto_escuro_area[0][0] <= center[0] <= self.cesto_escuro_area[1][0] and \
                 self.cesto_escuro_area[0][1] <= center[1] <= self.cesto_escuro_area[1][1]:
                self.resultado["cesto_escuro"] += 1

        return self.resultado


def main():
    video = cv2.VideoCapture('video.mp4')  # substitua 'video.mp4' pelo caminho do vídeo
    contador = OrangeCounter()
    while True:
        ret, frame = video.read()
        if not ret:
            break
        resultado = contador.run(frame)
        cv2.putText(frame, f"Cesto Claro: {resultado['cesto_claro']}", (10, 50),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)
        cv2.putText(frame, f"Cesto Escuro: {resultado['cesto_escuro']}", (10, 100),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)
        cv2.imshow('Video', frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
    video.release()
    cv2.destroyAllWindows()
    print(resultado)

if __name__ == "__main__":
    main()  # l4r4nj4




import cv2
import numpy as np

class OrangeCounter:
    def __init__(self):
        # Inicializamos o dicionário para contar as laranjas
        self.resultado = {"cesto_claro": 0, "cesto_escuro": 0}  # l4r4nj4
        # Guardar as últimas posições das laranjas
        self.laranjas_anteriores = []

    def detectar_cestos(self, frame):
        # Convertendo a imagem para HSV para facilitar a detecção das cores dos cestos
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        # Definir intervalos de cor para os cestos claro e escuro (exemplo, ajustar conforme o vídeo)
        lower_claro = np.array([0, 0, 200])  # Ajuste a cor do cesto claro
        upper_claro = np.array([180, 50, 255])
        lower_escuro = np.array([0, 0, 0])  # Ajuste a cor do cesto escuro
        upper_escuro = np.array([180, 50, 50])

        # Máscaras para detectar cestos claro e escuro
        mask_claro = cv2.inRange(hsv, lower_claro, upper_claro)
        mask_escuro = cv2.inRange(hsv, lower_escuro, upper_escuro)

        # Encontrando os contornos dos cestos
        contours_claro, _ = cv2.findContours(mask_claro, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        contours_escuro, _ = cv2.findContours(mask_escuro, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # Supondo que os maiores contornos sejam os cestos
        cesto_claro = max(contours_claro, key=cv2.contourArea) if contours_claro else None
        cesto_escuro = max(contours_escuro, key=cv2.contourArea) if contours_escuro else None

        # Retornando as posições dos cestos
        return cesto_claro, cesto_escuro

    def detectar_laranjas(self, frame):
        # Convertendo o frame para HSV
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        # Definir intervalo de cor para laranja
        lower_orange = np.array([5, 100, 100])
        upper_orange = np.array([15, 255, 255])

        # Máscara para detectar objetos laranjas
        mask = cv2.inRange(hsv, lower_orange, upper_orange)

        # Encontrando os contornos das laranjas
        contours, _ = cv2.findContours(mask, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # Retornar os contornos encontrados
        return contours

    def laranja_entrou_no_cesto(self, laranja, cesto):
        # Checar se o centro da laranja está dentro do cesto
        if cesto is not None:
            x, y, w, h = cv2.boundingRect(cesto)
            laranja_x, laranja_y, laranja_w, laranja_h = cv2.boundingRect(laranja)
            laranja_centro = (laranja_x + laranja_w // 2, laranja_y + laranja_h // 2)

            return x <= laranja_centro[0] <= x + w and y <= laranja_centro[1] <= y + h
        return False

    def run(self, frame):
        # Detectar os cestos no frame atual
        cesto_claro, cesto_escuro = self.detectar_cestos(frame)

        # Detectar as laranjas no frame atual
        laranjas = self.detectar_laranjas(frame)

        # Verificar se as laranjas saíram do frame, indicando que entraram em um cesto
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

        # Atualizar a lista de laranjas atuais
        self.laranjas_anteriores = novas_laranjas

        return self.resultado

def main():
    video = cv2.VideoCapture('video.mp4')  # Substitua 'video.mp4' pelo caminho do vídeo
    contador = OrangeCounter()

    while True:
        ret, frame = video.read()
        if not ret:
            break

        # Executa a contagem das laranjas
        resultado = contador.run(frame)

        # Escreve o resultado na imagem
        cv2.putText(frame, f"Cesto Claro: {resultado['cesto_claro']}", (10, 50),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)
        cv2.putText(frame, f"Cesto Escuro: {resultado['cesto_escuro']}", (10, 100),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2, cv2.LINE_AA)

        # Mostra o vídeo com os resultados
        cv2.imshow('Video', frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    video.release()
    cv2.destroyAllWindows()

    # Imprime o resultado final
    print(resultado)

if __name__ == "__main__":
    main()  # l4r4nj4



