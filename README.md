# Zsh Command Not Found Handler

This script adds a custom command_not_found_handler function to your `.zshrc` file. When you try to execute a command that is not found, it will provide suggestions for installing the package that contains that command (if available).

## Usage

1. Run the following command in your terminal:

```bash
cat << 'EOF' >> ~/.zshrc
# Add command_not_found_handler function
function command_not_found_handler {
    local output=$(/usr/lib/command-not-found -- "$1" 2>&1)
    local cmd_status=$?

    RED='\033[0;31m'
    GREEN='\033[0;32m'
    YELLOW='\033[0;33m'
    BLUE='\033[0;34m'
    NC='\033[0m'

    if [ $cmd_status -eq 0 ]; then
        echo -e "${NC}$output${NC}"
        local install_command=$(echo "$output" | grep -oE 'sudo (apt install|snap install) [a-zA-Z0-9\-]+')

        if [[ -n "$install_command" ]]; then
            echo -en "${YELLOW}🔍 Do you want to execute: 📦 ${RED}$install_command${YELLOW} (y/N) ${NC}"
            read -r response
            case $response in
                [Yy]* )
                    echo -ne "${BLUE}⚡ Installing... ["
                    if eval "$install_command" > /dev/null 2>&1; then
                        local start_time=$(date +%s)
                        while [ $(($(date +%s) - start_time)) -lt 2 ]; do
                            echo -ne "▉"
                            sleep 0.2
                        done
                        echo -e "]${NC}"
                        echo -e "${GREEN}✅ Installation completed successfully!${NC}"
                    else
                        echo -e "]${NC}"
                        echo -e "${RED}❌ Installation failed. Please check the error messages above.${NC}"
                    fi
                    ;;
                * )
                    echo -e "${RED}💥 Installation aborted by the user.${NC}"
                    ;;
            esac
        else
            echo -e "${RED}🔍 Package not found!${NC}"
        fi
    else
        echo -e "${RED}$output${NC}"
    fi

    return $cmd_status
}
EOF
```
Restart your shell or run source ~/.zshrc to apply the changes.

## License
This project is licensed under the MIT License - see the LICENSE file for details.