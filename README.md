# teste

import cv2

class OrangeCounter:
    def __init__(self):
        self.resultado = {"cesto_claro": 0, "cesto_escuro": 0}  # l4r4nj4

    def run(self, frame):
        # Aqui você pode adicionar a lógica de processamento de imagem para contar as laranjas
        # e atualizar o dicionário 'resultado'.
        # Como exemplo, o código está apenas retornando o dicionário original.
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