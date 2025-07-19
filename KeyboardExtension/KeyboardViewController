import UIKit
import AudioToolbox

class KeyboardViewController: UIInputViewController {
    
    // MARK: - Properties
    private let feedbackGenerator = UIImpactFeedbackGenerator(style: .rigid)
    private var shiftOn = false
    private var keys: [UIButton] = []
    
    // REVERSED intensity map (rare keys = strongest feedback)
    private let intensityMap: [String: CGFloat] = [
        "z": 1.0, "x": 0.95, "q": 0.90, "j": 0.85,
        "return": 0.82, "k": 0.80, "123": 0.78, "emoji": 0.75,
        "v": 0.72, "b": 0.70, "y": 0.65, "p": 0.62,
        "g": 0.60, "f": 0.58, "w": 0.55, "m": 0.52,
        "u": 0.50, "c": 0.48, "shift": 0.45, "l": 0.42,
        "d": 0.40, "r": 0.38, "h": 0.35, "backspace": 0.33,
        "s": 0.30, "n": 0.28, "i": 0.25, "o": 0.22,
        "a": 0.20, "t": 0.18, "e": 0.15, "space": 0.10
    ]
    
    // Sound IDs
    private let pluckSound: SystemSoundID = 1105
    private let thudSound: SystemSoundID = 1156
    
    // MARK: - View Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        setupKeyboard()
        feedbackGenerator.prepare()
    }
    
    // MARK: - Keyboard Setup
    private func setupKeyboard() {
        view.subviews.forEach { $0.removeFromSuperview() }
        keys.removeAll()
        
        let containerView = UIView()
        containerView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(containerView)
        
        NSLayoutConstraint.activate([
            containerView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            containerView.topAnchor.constraint(equalTo: view.topAnchor),
            containerView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            containerView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        
        let stackView = UIStackView()
        stackView.axis = .vertical
        stackView.spacing = 6
        stackView.translatesAutoresizingMaskIntoConstraints = false
        containerView.addSubview(stackView)
        
        NSLayoutConstraint.activate([
            stackView.leadingAnchor.constraint(equalTo: containerView.leadingAnchor, constant: 4),
            stackView.trailingAnchor.constraint(equalTo: containerView.trailingAnchor, constant: -4),
            stackView.topAnchor.constraint(equalTo: containerView.topAnchor, constant: 8),
            stackView.bottomAnchor.constraint(equalTo: containerView.safeAreaLayoutGuide.bottomAnchor, constant: -8)
        ])
        
        stackView.addArrangedSubview(createRow(letters: ["q", "w", "e", "r", "t", "y", "u", "i", "o", "p"]))
        stackView.addArrangedSubview(createRow(letters: ["a", "s", "d", "f", "g", "h", "j", "k", "l"]))
        stackView.addArrangedSubview(createRow(letters: ["shift", "z", "x", "c", "v", "b", "n", "m", "backspace"]))
        stackView.addArrangedSubview(createSpecialRow())
    }
    
    private func createRow(letters: [String]) -> UIStackView {
        let rowStack = UIStackView()
        rowStack.distribution = .fillEqually
        rowStack.spacing = 4
        
        for letter in letters {
            let button = createKeyButton(title: letter)
            rowStack.addArrangedSubview(button)
            keys.append(button)
        }
        return rowStack
    }
    
    private func createSpecialRow() -> UIStackView {
        let rowStack = UIStackView()
        rowStack.distribution = .fill
        rowStack.spacing = 4
        
        let keys = [
            createKeyButton(title: "123"),
            createKeyButton(title: "emoji"),
            createKeyButton(title: "space"),
            createKeyButton(title: "return")
        ]
        
        keys.forEach {
            rowStack.addArrangedSubview($0)
            self.keys.append($0)
        }
        
        // Set spacebar to take 50% width
        if let spaceIndex = keys.firstIndex(where: { $0.title(for: .normal) == "space" }) {
            keys[spaceIndex].widthAnchor.constraint(equalTo: rowStack.widthAnchor, multiplier: 0.5).isActive = true
        }
        
        return rowStack
    }
    
    private func createKeyButton(title: String) -> UIButton {
        let button = UIButton(type: .system)
        button.setTitle(title, for: .normal)
        button.titleLabel?.font = UIFont.systemFont(ofSize: 18, weight: .medium)
        button.backgroundColor = .secondarySystemBackground
        button.layer.cornerRadius = 5
        button.layer.masksToBounds = true
        
        button.addTarget(self, action: #selector(keyPressed(_:)), for: .touchUpInside)
        button.addTarget(self, action: #selector(keyTouchDown(_:)), for: .touchDown)
        button.addTarget(self, action: #selector(keyTouchUp(_:)), for: [.touchUpInside, .touchCancel])
        
        return button
    }
    
    // MARK: - Key Actions
    @objc private func keyTouchDown(_ sender: UIButton) {
        guard let keyId = sender.title(for: .normal)?.lowercased() else { return }
        
        let intensity = intensityMap[keyId] ?? 0.5
        triggerHaptic(intensity: intensity)
        
        // Play sounds
        if intensity >= 0.7 {
            AudioServicesPlaySystemSound(pluckSound)
        } else if intensity >= 0.4 {
            AudioServicesPlaySystemSound(thudSound)
        }
        
        // Visual feedback
        sender.backgroundColor = .systemBlue.withAlphaComponent(min(0.7, (1.3 - intensity) * 0.5))
    }
    
    @objc private func keyTouchUp(_ sender: UIButton) {
        UIView.animate(withDuration: 0.1) {
            sender.backgroundColor = .secondarySystemBackground
        }
    }
    
    @objc private func keyPressed(_ sender: UIButton) {
        guard let keyId = sender.title(for: .normal)?.lowercased() else { return }
        
        switch keyId {
        case "space":
            textDocumentProxy.insertText(" ")
        case "backspace":
            textDocumentProxy.deleteBackward()
        case "shift":
            shiftOn.toggle()
            toggleShiftState()
        case "return":
            textDocumentProxy.insertText("\n")
        case "123", "emoji":
            advanceToNextInputMode()
        default:
            if keyId.count == 1 {
                let char = shiftOn ? keyId.uppercased() : keyId
                textDocumentProxy.insertText(char)
                if shiftOn {
                    shiftOn = false
                    toggleShiftState()
                }
            }
        }
    }
    
    private func toggleShiftState() {
        keys.forEach {
            guard let title = $0.title(for: .normal), title.count == 1 else { return }
            $0.setTitle(shiftOn ? title.uppercased() : title.lowercased(), for: .normal)
        }
        
        if let shiftButton = keys.first(where: { $0.title(for: .normal) == "shift" }) {
            shiftButton.backgroundColor = shiftOn ? .systemBlue : .secondarySystemBackground
            shiftButton.setTitleColor(shiftOn ? .white : .label, for: .normal)
        }
    }
    
    private func triggerHaptic(intensity: CGFloat) {
        guard !UIAccessibility.isReduceMotionEnabled else { return }
        feedbackGenerator.impactOccurred(intensity: pow(intensity, 1.5))
        feedbackGenerator.prepare()
    }
    
    // MARK: - Required Overrides
    override func updateViewConstraints() { super.updateViewConstraints() }
    override func textWillChange(_ textInput: UITextInput?) {}
    override func textDidChange(_ textInput: UITextInput?) {}
}
